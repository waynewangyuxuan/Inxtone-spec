# Progress Archive

> Sessions before 2026-02-10 — M1 through M3.5

**Parent**: [Progress/Meta.md](Meta.md)

---

## 2026-02-09 (M3.5: Hackathon Submission)

### Completed
- **Phase 1: English AI Prompts** — Translated all 5 prompt templates and context builder labels
- **Phase 2: Per-Request API Key (BYOK)** — Client sends `X-Gemini-Key` header. GeminiProvider per-call instance. `POST /api/ai/verify-key`
- **Phase 3: API Key Dialog** — Zustand store, modal with masked key, verify flow, skip option
- **Phase 4: Seed Loader** — Raw SQL seed files (`sql-en.ts`, `sql-zh.ts`), bundled by tsup. Endpoints: status/load/clear
- **Phase 5: Welcome Screen & Settings** — 3-card welcome (EN/ZH Demo, Start Empty)
- **Phase 6: Deployment** — Multi-stage Dockerfile
- **Phase 7: Submission Materials**

Stats: **1001 tests** | 10 new files, 12 modified

---

## 2026-02-08 (M3 Phases 2-5)

### M3 Phase 5: Plot UI
- Standalone `/plot` page with 3 tabs (Arcs, Foreshadowing, Hooks)
- Arc Outliner, Foreshadowing Tracker (timeline + add hint + resolve/abandon), Hook Tracker
- 10 new files, 4 modified

### M3 Phase 4: Chapter Editor UI
- Three-panel layout (chapters/bible | editor | AI sidebar)
- Foundation: useChapters hooks, useEditorStore, aiStream SSE utility
- ChapterListPanel, StoryBiblePanel, ChapterForm, EditorPanel, EditorToolbar
- AISidebar, ContextPreview, StreamingResponse, RejectReasonModal
- 19 new files, 3 modified | 984 tests

### M3 Phase 3: Writing API Routes
- 18 endpoints across volumes/chapters/versions/stats
- 9 Zod schemas, validateBody/validateQuery helpers
- Server bootstrap with WritingService DI
- 39 integration tests | **976 tests** total

### M3 Phase 2: AI Service
- GeminiProvider (streaming, retry, error mapping)
- ContextBuilder (5-layer FK-based, token budget management)
- PromptAssembler (YAML front-matter + {{variable}} substitution)
- AIService (6 generation methods, EventBus integration, monitoring)
- SSE API Routes (6 streaming + 2 JSON endpoints)
- 109 new tests | **936 tests** total

---

## 2026-02-07 (M2 Complete + M3 Phase 1)

### M3 Phase 1: Writing Service
- WritingRepository (738 lines): Volume/Chapter CRUD, content ops, version management, FK cleanup
- WritingService (650 lines): 25 methods, FK validation, status state machine, EventBus
- 127 new tests | **818 tests** total

### M2 Phase 6: Testing & Polish
- CLI Bible command tests (17), performance benchmarks (12)
- **695 tests**, all perf targets met (sub-ms operations)
- M2 Complete: 6 phases, 45 REST endpoints, full Web UI, 3 CLI commands

### Demo Seed Data
- 《墨渊记》Ink Abyss Chronicles — 6 characters, 7 relationships, world settings, 4 locations, 3 factions, 5 timeline events, 2 arcs, 3 foreshadowing, 4 hooks

### M2 Phase 5: CLI Commands
- `inxtone bible list/show/search` — 524-line implementation with rich formatting

### M2 Phase 4: Web UI Fixes
- 6 critical fixes (forms, error handling, window.prompt removal)
- 5 high priority fixes (API client, cache, delete invalidation)
- 40+ files modified

### M2 Phases 1-3
- **Phase 3: API Layer** — 45 endpoints, 76 route tests, 666 tests total
- **Phase 2: Service Layer** — EventBus + StoryBibleService (41 methods), 590 tests
- **Phase 1: Repository Layer** — 7 repositories, 341 tests

---

## 2026-02-06

- M1 sign-off — merged PR #1 (ms1 -> main)
- Milestone roadmap planning (M2-M5)

---

## 2026-02-05 (Sessions 1-5)

### Session 4-5: M1 Phase 5 + 6
- CLI Shell: Commander.js, `inxtone init`, `inxtone serve`
- Server + Web Shell: static serving, SPA fallback, CSS design system, AppShell layout, React Router

### Session 3: M1 Phase 4
- Database Schema: SQLite, WAL mode, MigrationRunner, 18 core tables, FTS5
- 238 tests passing

### Session 2: M1 Phase 1 + 2
- Project setup: pnpm monorepo, TypeScript strict, Vitest
- Interface contract: all entity types, service interfaces, event system
- ESLint v9, pre-commit hooks

### Session 1: Project Init
- Meta/ folder hierarchy, ADR system, Milestone tracking, CLAUDE.md
- ADR-0001: TypeScript/Node.js tech stack
