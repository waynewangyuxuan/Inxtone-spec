# AIService 模块设计

> AI 调用的核心模块：Context 构建、Gemini 调用、流式输出

---

## 一、模块职责

| 子功能 | 说明 |
|--------|------|
| **GeminiProvider** | Gemini 2.5 Pro 调用 (`@google/genai` SDK) |
| **ContextBuilder** | 基于 Chapter 外键的确定性 context 组装 (核心大脑) |
| **PromptAssembler** | 模板化的提示词系统 |
| **流式输出** | SSE 实时返回生成内容 |
| **错误处理** | 重试、Rate Limit 处理 |

> **MVP 决策**: 只支持 Gemini 2.5 Pro，不做 Provider 抽象和 fallback。
> 多 Provider 支持 (Claude, OpenAI) 推迟到 M4+。

> **模型时效性说明 (2026-07-01)**: 本文档中的 "Gemini 2.5 Pro" / `gemini-2.5-pro` 是 M3 时期的历史决策记录。当前默认模型为 `gemini-3.5-flash`（gemini-2.0/2.5 系列已被 Google 陆续关停）。**模型 ID 应收敛于代码配置单点，文档不再逐处追更具体型号**。Context 组装策略的演进见 ADR-0009。

---

## 二、核心工作流

### 2.1 AI 续写章节

```
用户操作: 在编辑器中按 Ctrl+A，选择 "续写"
    │
    ▼
┌─────────────────────────────────────────┐
│ 1. 收集用户输入                          │
│    - 当前章节内容                        │
│    - 续写长度偏好 (短/中/长)             │
│    - 风格偏好 (简洁/详细)                │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│ 2. AIService.continueScene(input)       │
│    │                                     │
│    │  ┌────────────────────────────────┐│
│    │  │ 2.1 构建 Context               ││
│    │  │     ContextBuilder.build()     ││
│    │  └────────────────────────────────┘│
│    │                 │                   │
│    │                 ▼                   │
│    │  ┌────────────────────────────────┐│
│    │  │ 2.2 选择 Prompt 模板           ││
│    │  │     prompts/continue.md        ││
│    │  └────────────────────────────────┘│
│    │                 │                   │
│    │                 ▼                   │
│    │  ┌────────────────────────────────┐│
│    │  │ 2.3 组装最终 Prompt            ││
│    │  │     {context} + {template}     ││
│    │  │     + {current_content}        ││
│    │  └────────────────────────────────┘│
│    │                 │                   │
│    │                 ▼                   │
│    │  ┌────────────────────────────────┐│
│    │  │ 2.4 调用 Provider              ││
│    │  │     provider.stream(prompt)    ││
│    │  └────────────────────────────────┘│
│    │                                     │
└─────────────────────────────────────────┘
    │
    ▼ (流式返回)
┌─────────────────────────────────────────┐
│ 3. 前端逐字显示                          │
│    - 生成中: 所有 AI 按钮禁用           │
│    - 逐 chunk 追加到预览区               │
│    - 流中断 → 保留已收到的部分内容      │
│    - 显示 Token 使用量                   │
└─────────────────────────────────────────┘
    │
    ▼ (生成完成 或 流中断)
┌─────────────────────────────────────────┐
│ 4. 用户决策                              │
│    ┌─────────┐ ┌─────────┐ ┌─────────┐ │
│    │  采纳   │ │ 重新生成 │ │  拒绝   │ │
│    └────┬────┘ └────┬────┘ └────┬────┘ │
│         │           │           │       │
│    插入到光标  复用context+   必须填写   │
│    位置       注入reject     拒绝理由   │
│    (含半截)   reason重新生成            │
└─────────────────────────────────────────┘
```

### 2.2 ContextBuilder 构建流程 (核心大脑)

> **关键设计变更**: MVP 不使用语义搜索。改为基于 Chapter 实体外键的**确定性 context 组装**。
> Chapter.characters[], Chapter.locations[], Chapter.foreshadowingHinted[] 等外键提供
> 精确的、作者意图驱动的 context。语义搜索 (M4) 将作为增强，而非替代。

```
输入: chapterId + userInstruction
    │
    ▼
┌─────────────────────────────────────────┐
│ 1. 计算可用 Token 预算                   │
│                                         │
│    总预算 = 1,000,000 (Gemini 2.5 Pro) │
│           - reserveForOutput (4000)     │
│           - reserveForPrompt (2000)     │
│                                         │
│    可用: ~994,000 tokens               │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│ 2. 分层收集 (Layer-based Assembly)      │
│                                         │
│ Layer 1 — Required (必须):             │
│    a. 当前章节 content                  │
│    b. 当前章节 outline (goal, scenes)  │
│    c. 前一章末尾 500 字                 │
│                                         │
│ Layer 2 — FK Expansion (批量查询):     │
│    d. chapter.characters[]              │
│       → CharacterRepo.findByIds([...]) │
│       (name, appearance, motivation,   │
│        facets, voiceSamples)           │
│    e. chapter.locations[]               │
│       → LocationRepo.findByIds([...]) │
│    f. chapter.arcId → Arc 结构+进度     │
│    g. Scoped Relationships:            │
│       → 仅查本章角色间的直接关系        │
│       → A↔B 有直接关系 → 包含          │
│       → A↔B 无直接关系但 A→C→B → 包含  │
│       → 不拉入章外角色 C 的完整档案     │
│                                         │
│ Layer 3 — Plot Awareness (批量查询):   │
│    h. chapter.foreshadowingHinted[]    │
│       → ForeshadowingRepo.findByIds() │
│    i. 当前 Arc 的 active foreshadowing │
│    j. 上一章的 hook → 确保连贯         │
│                                         │
│ Layer 4 — World Rules (世界规则):      │
│    k. powerSystem.coreRules (总是包含) │
│    l. socialRules (如果相关)           │
│                                         │
│ Layer 5 — User-Selected (用户选择):    │
│    m. 用户在 UI 中手动添加的内容       │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│ 3. 按优先级填充，直到达到预算            │
│                                         │
│    优先级: L1 > L2 > L3 > L4 > L5     │
│    超出预算时从低优先级开始裁剪         │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│ 4. 格式化输出                            │
│                                         │
│    <context>                            │
│    ## 角色档案                          │
│    ### 林逸                             │
│    {完整档案: 外貌/动机/性格面}         │
│    ### 陈浩                             │
│    {完整档案}                           │
│                                         │
│    ## 角色关系 (仅本章角色间)            │
│    林逸 ↔ 陈浩: 宿敌 (始于Ch.3的羞辱)   │
│                                         │
│    ## 场景地点                          │
│    宗门擂台: {描述, 氛围}               │
│                                         │
│    ## 剧情提醒                          │
│    ⚠ 本章需暗示伏笔: 老爷子的真实身份   │
│    📌 当前 Arc 进度: 73% (天才对决)     │
│                                         │
│    ## 世界规则                          │
│    {力量体系核心规则}                    │
│                                         │
│    ## 前文                              │
│    {前一章末尾}                         │
│                                         │
│    ## 本章大纲                          │
│    {goal, scenes, hookEnding}           │
│    </context>                           │
└─────────────────────────────────────────┘
```

### 2.3 Gemini Provider 调用

> **MVP 决策**: 单 Provider (Gemini 2.5 Pro)，不做 fallback chain。
> Provider 抽象层保留在接口定义中 (services.ts)，但 M3 只实现 GeminiProvider。

```
配置:
┌─────────────────────────────────────────┐
│ ai_config:                               │
│   provider: gemini                       │
│   model: gemini-2.5-pro                 │
│   sdk: @google/genai                    │
│   temperature: 0.7                      │
│   max_output_tokens: 4000               │
│   max_input_tokens: 1_000_000           │
│   retry_count: 3                        │
│   retry_delay: 1000ms                   │
└─────────────────────────────────────────┘

调用流程:
    │
    ▼
┌─────────────────────────────────────────┐
│ try:                                    │
│   for attempt in 1..retry_count:       │
│     result = gemini.generateContent-   │
│              Stream(prompt)            │
│     if success: stream result          │
│     if RateLimit: wait retry-after     │
│     if Timeout: retry with backoff     │
│                                         │
│ catch AuthError:                        │
│   → 提示用户检查 API Key              │
│                                         │
│ catch ContentFilter:                    │
│   → 提示内容被过滤，建议修改输入      │
│                                         │
│ catch NetworkError:                     │
│   → 重试，超过次数后提示用户          │
└─────────────────────────────────────────┘
```

---

## 2.4 四种生成模式

```
┌─────────────┬──────────────────────────────────────────┐
│ Continue    │ 续写接下来的完整内容 (核心功能)           │
│ (续写)      │ Accept → 插入到编辑器光标位置            │
├─────────────┼──────────────────────────────────────────┤
│ Dialogue    │ 只生成一段对话，专注角色交互              │
│ (对话)      │ Accept → 插入到编辑器光标位置            │
├─────────────┼──────────────────────────────────────────┤
│ Describe    │ 只生成一段描写 (场景/人物/动作)          │
│ (描写)      │ Accept → 插入到编辑器光标位置            │
├─────────────┼──────────────────────────────────────────┤
│ Brainstorm  │ 给用户一个续写概念/方向预览              │
│ (头脑风暴)  │ 满意 → 基于概念执行 Continue 续写        │
│             │ 不满意 → Reject + 理由 → Regenerate     │
└─────────────┴──────────────────────────────────────────┘
```

### 2.5 Reject + Regenerate 流程

```
[Reject] 点击:
  │
  ▼
弹出: "为什么不满意？" (必填理由)
  │
  ▼
保存 { rejectedContent, rejectReason, generationType }
  │
  ▼
[Regenerate] 点击:
  │
  ├─ 默认: 复用当前 context (不重新 build)
  │
  ├─ Prompt 注入:
  │   "上次生成被拒绝。
  │    原因: {{rejectReason}}
  │    被拒内容: {{rejectedContent}}
  │    请避免相同问题。"
  │
  └─ MVP: 始终复用当前 context (不做 AI 判断)
      未来增强 (M4+): AI 自行判断是否需要补充
```

---

## 三、Prompt 模板系统

```
目录结构:
prompts/
├── continue.md        # 续写
├── dialogue.md        # 对话生成
├── describe.md        # 场景描写
├── brainstorm.md      # 头脑风暴
├── ask_bible.md       # Story Bible 问答
├── check/
│   ├── consistency.md # 一致性检查
│   └── wayne.md       # Wayne 原则检查
└── design/
    ├── character.md   # 角色设计
    └── plot.md        # 剧情设计

模板示例 (continue.md):
┌─────────────────────────────────────────┐
│ ---                                     │
│ name: 续写                              │
│ description: 续写当前章节内容           │
│ variables:                              │
│   - context                             │
│   - current_content                     │
│   - style_preference                    │
│ ---                                     │
│                                         │
│ 你是一位网文写作助手。                  │
│                                         │
│ {{context}}                             │
│                                         │
│ ## 当前内容                             │
│ {{current_content}}                     │
│                                         │
│ 请续写接下来的内容。要求:               │
│ - 保持角色性格一致                      │
│ - 风格: {{style_preference}}            │
│ - 自然衔接，不要重复已有内容           │
│                                         │
│ 直接输出续写内容，无需解释。           │
└─────────────────────────────────────────┘
```

---

## 四、数据流向

```
┌─────────────────────────────────────────────────────────┐
│                     用户界面                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │  AI 面板    │  │  续写按钮   │  │  Story Bible问答│ │
│  └──────┬──────┘  └──────┬──────┘  └────────┬────────┘ │
└─────────┼────────────────┼──────────────────┼──────────┘
          │                │                  │
          └────────────────┴──────────────────┘
                           │
                   ┌───────┴───────┐
                   │   AIService   │
                   └───────┬───────┘
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Context     │    │  Prompt     │    │  Gemini     │
│ Builder     │    │  Assembler  │    │  Provider   │
│ (FK-based)  │    │             │    │  (2.5 Pro)  │
└──────┬──────┘    └─────────────┘    └─────────────┘
       │
       ▼
┌─────────────┐
│ StoryBible  │
│ Service     │ ←── 通过外键查找角色/地点/伏笔
└─────────────┘
```

---

## 五、关键设计决策

### 5.1 为什么用 SSE 而非 WebSocket 做流式输出

```
SSE (Server-Sent Events):
  ✓ 单向：服务器 → 客户端（正是我们需要的）
  ✓ 基于 HTTP，无需额外协议
  ✓ 自动重连
  ✓ 浏览器原生支持

WebSocket:
  - 双向通信（AI 输出不需要）
  - 需要额外连接管理
  - 复杂度更高

选择: SSE 用于 AI 流式输出
     WebSocket 用于其他实时同步（如协作编辑）
```

### 5.2 Context 优先级算法

> **MVP 变更**: 不使用 semantic_relevance 评分 (M4 加入)。
> 改为基于 Layer 的确定性优先级。

```
Layer 优先级 (高→低):
  Layer 1 (Required):      priority = 1000
  Layer 2 (FK Expansion):  priority = 800
  Layer 3 (Plot Awareness): priority = 600
  Layer 4 (World Rules):   priority = 400
  Layer 5 (User-Selected): priority = 200

Token 预算不足时，从 Layer 5 开始裁剪。
Layer 1 永远不被裁剪。

未来增强 (M4+):
  + semantic_relevance * 100  (语义搜索加分)
  + recency_score * 50        (近期使用加分)
```

### 5.3 Token 计数策略

```
精确计数 (生产环境):
  - 使用 tiktoken (OpenAI) 或对应的 tokenizer
  - 每个 Provider 用自己的 tokenizer

估算计数 (开发/快速):
  - 中文: 字数 * 1.5
  - 英文: 单词数 * 1.3

选择:
  - Context 构建时用估算（速度快）
  - 提交前用精确计数验证
```

---

## 六、错误处理

| 错误场景 | 处理 |
|----------|------|
| Rate Limit | 等待 retry-after 秒后重试 |
| Token Limit 超出 | 自动裁剪 Context，保留高优先级 |
| Network Timeout | 重试 3 次，间隔递增 |
| Auth Error | 提示用户检查 API Key |
| Content Filter | 提示内容被过滤，建议修改输入 |
| Gemini 不可用 | 提示网络问题，建议稍后重试 |

---

## 七、监控指标

| 指标 | 用途 |
|------|------|
| `ai.request.count` | 请求次数 |
| `ai.request.latency` | 响应时间 |
| `ai.token.input` | 输入 Token 数 |
| `ai.token.output` | 输出 Token 数 |
| `ai.error.rate` | 错误率 |
---

## 八、相关决策

- **ADR-0002**: AI Context Injection Strategy (L1-L5 分层设计)
- **ADR-0004**: Write Page Bible Panel Entity Management & Context Auto-Rebuild
  - 章节 FK 数组更新后自动触发 Context rebuild
  - Outline 保存后自动触发 Context rebuild

---

*Review 重点: ContextBuilder 分层组装、批量查询策略、Scoped Relationships、Token 预算管理、SSE 流式输出*
*MVP 决策: Gemini 2.5 Pro only, FK-based context (no semantic search), Sidebar Preview + Accept, Manual Save*
