# ADR-0008: Design Language — Paper-Ink-Cinnabar with Hand-Drawn Texture

- **Status**: Proposed
- **Date**: 2026-07-01
- **Deciders**: Wayne

## Context

A full UI audit (2026-07-01, code + live screenshots of 7 pages) confirmed Wayne's verdict ("难看、没有 taste、cliché、文字可读性差"). Two root findings:

**1. The current theme is a named cliché**: "Premium Dark SaaS Dashboard with Gold Accent" — near-black base, gold accent, thin-weight display headings, uppercase micro-labels, glowing card borders. The 2023–25 standard look of crypto dashboards. `tokens.css` even self-describes as "premium writing experience". Information architecture follows suit: the entry page is six KPI stat cards (a story's *monitoring system*, not a story's *study*); Story Bible is CRUD-table grammar; Chinese character names sit beside "SUPPORTING / MOTIVATION LAYERS" uppercase English badges.

**2. Chinese readability failure has concrete technical causes**:
- Font stack contains **no Chinese font**; the declared Inter is **never actually loaded** (no @font-face, no link — verified via `document.fonts`)
- Global `font-weight: 300` — thin-weight CJK strokes blur on near-black
- Body text 14px (below the CJK floor of 15–16px), including the editor where authors write 100k字
- `font-style: italic` applied to Chinese (no italic glyphs exist; browsers mechanically slant)
- `<html lang="en">` causes wrong Han glyph variant selection
- Contrast: `--gray-dark #666` on `#0a0a0a` ≈ 3.9:1, below WCAG AA at small sizes

Wayne's stated taste: pencil/crayon texture, pure black-and-white primary palette with sparse accents, Excalidraw-like hand-drawn line quality, ruthless simplicity, first screen must self-explain.

## Decision

Adopt the design language **「纸墨朱砂 × 铅笔手绘」(Paper–Ink–Cinnabar with hand-drawn texture)** — Wayne's pencil/B&W taste supplies the *stroke and texture*; the inkstone metaphor supplies the *ground and identity*:

### Palette
- **Ground**: warm paper white `#F7F4ED` (宣纸), not pure white; layering by whitespace, not card borders
- **Ink five-step scale replaces the gray system** (焦墨 `#1C1A17` → 浓墨 → 淡墨 → 清墨 → 水痕 `#E5E0D4`): all text hierarchy expressed as ink dilution — this *is* the "pure black-and-white primary"
- **Cinnabar red (~`#A63A2E`) is the single accent** — the 朱笔 (annotation brush): primary actions, current selection, and the visual identity of AI suggestions. Gold demoted to milestone moments only
- **Dark mode is "ground ink," not black**: `#211E1A` warm black + `#EAE4D6` mi-white — a desk lamp over manuscript paper, not a server room

### Hand-drawn texture (the Excalidraw layer)
Borders, dividers, relationship-graph edges, and the chapter timeline render with hand-drawn/pencil stroke quality (rough edges, slight waver). This converts the visualizations from "d3 demo with neon defaults" into "the author's own story sketch." Texture lives in *lines*, not in fills — restraint keeps it a tool, not a toy.

### Typography (binding rules)
```css
--font-ui:    "Inter", "Noto Sans SC", "PingFang SC", system-ui, sans-serif;
--font-prose: "Source Serif 4", "Noto Serif SC", "Songti SC", serif;  /* editor + Bible content */
```
- Fonts must be **actually bundled** (self-hosted subsets; local-first compatible) — no more phantom stacks
- CJK weight always ≥ 400; hierarchy via size and ink dilution, never thin weights
- **No italics on Chinese** — emphasis via cinnabar color or `text-emphasis: dot` (着重号)
- Editor prose: 17–18px, line-height ~1.9, measure 32–38em; UI base 15px in mixed-CJK contexts; `<html lang="zh-CN">`

### Three interface transformations (direction, not spec)
1. **案头 (entry page)**: the *work itself* replaces the KPI dashboard — title in serif, total words rendered as ink-pool level rising in an inkstone; a seal (印章) stamped per 100k字 as the achievement system. Stats demoted to one small line.
2. **墨线章回 (chapter spine)**: chapters as ink dots on a growing vertical stroke (size = word count, dilution = status); foreshadowing as thin branches leaving the spine and rejoining (resolved) or yellowing (overdue) — "watching the story grow" made literal. Replaces Plot page's Arcs/Foreshadowing tabs.
3. **批注式 AI 栏 (annotation margin)**: writing page defaults to *paper only* (nav collapsed to a 48px spine, AI panel closed); AI suggestions appear as cinnabar margin annotations with 落笔/另想/搁置 actions; accepting triggers a restrained ink-bleed animation at the insertion point. Markdown toolbar removed from the prose editor.

### Immediate triage list (ships before any redesign; ~1 day)
1. Bundle Noto Sans SC subset; add CJK fonts to stack; `lang="zh-CN"`
2. Global weight 300 → 400; body 14px → 15/16px
3. Remove italics from Chinese text (regular + emphasis mark or color)
4. API-key modal: persist "skip"; demote to a passive banner
5. Merge the duplicate token systems (`--text-*`/`--font-*`, `--space-sm`/`--space-2`)

## Alternatives Considered

### Alternative 1: Pure Excalidraw hand-drawn skin (white bg, sketch everything)
- **Pros**: Matches Wayne's stated reference most literally; playful.
- **Cons**: Whiteboard aesthetics read as "diagram tool," not "writing desk"; no cultural identity for a product named 砚台; hand-drawn *fills* at scale become noisy in a text-dense app.
- **Why not**: Adopted as the *stroke layer* instead — texture in lines, paper-ink as the ground.

### Alternative 2: Pure literati ink-wash theme (水墨 skin)
- **Pros**: On-brand metaphor.
- **Cons**: 水墨 skins are their own cliché (tea-house websites); risks costume over function.
- **Why not**: Keep the *material world* of the inkstone (paper, ink, cinnabar) but reject pictorial ink-wash decoration.

### Alternative 3: Iterate the existing dark-gold theme
- **Pros**: Least work.
- **Cons**: The cliché is structural (palette + IA + typography all wrong for the audience); polishing it deepens the wrong identity.
- **Why not**: Diagnosis showed root causes, not surface bugs.

## Consequences

### Positive
- Readability fixed at the root (fonts/weight/size/lang) — 70% of the complaint resolved by the triage list alone
- A recognizable identity no competitor occupies; the codex view (ADR-0007's brand surface) gets its face
- The game-feel mandate (story growing, seals, ink pool) gets concrete mechanisms rather than gamification chrome

### Negative
- Full light-mode inversion touches every component (the current system is dark-only); staged migration required
- Bundled CJK fonts add ~1–2MB (subset) to the package
- Hand-drawn rendering for graph/timeline requires custom drawing work (e.g., rough.js-style) beyond stock d3

### Risks
- Taste is subjective and unvalidated: mitigate by shipping a tokens.css + one-page demo for Wayne's sign-off before component migration.

## Related

- [ADR-0007](ADR-0007-product-repositioning-okb-dual-host.md) — the codex view this language dresses
- spec/Design/ — existing design docs to be revised once this ADR is Accepted
- spec/Todo.md — triage list tracked as tasks
