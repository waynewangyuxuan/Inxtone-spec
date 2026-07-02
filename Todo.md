# Todo

> Deferred tasks and ideas - prioritized backlog

---

## High Priority (2026-07 Redirection — see ADR-0007/0008/0009)

- [ ] **UI triage list** (ADR-0008, ~1 day total, ships before redesign):
  - [ ] Bundle Noto Sans SC subset + add CJK fonts to stack + `lang="zh-CN"`
  - [ ] Global font-weight 300 → 400; body text 14px → 15/16px
  - [ ] Remove italics from Chinese text (use emphasis mark or cinnabar color)
  - [ ] API-key modal: persist "skip", demote to passive banner
  - [ ] Merge duplicate token systems (`--text-*`/`--font-*`, `--space-sm`/`--space-2`)
- [ ] **`@inxtone/mcp` Phase 1** (ADR-0007) — MCP server (CRUD + opinionated queries) + skill pack + `inxtone view`
- [ ] **Design language rollout** (ADR-0008) — new tokens.css + one-page demo for sign-off, then staged migration
- [ ] **Fastify 5 migration** — done, PR #23 awaiting review/merge

## High Priority (Deferred from M4)

- [ ] **i18n system setup** — react-i18next + translation files; 36 preset strings are English-only
- [x] **Web package test infra** — jsdom + @testing-library/react *(done in PR #24)*
## High Priority (M4 Phase 7 — Code Review Fixes)

**Must Fix**
- [x] **useAutoSave race condition** — capture chapterId at debounce start *(fixed in 9fb4ee0; network-layer abort completed in PR #24)*

**Should Fix**
- [x] **Auto-save collision** — clear pending timer + AbortController *(fixed in 9fb4ee0 + PR #24: signal now reaches fetch)*
- [x] **useAutoSave unit tests** — 6 unit tests *(done in PR #24)*
- [ ] **i18n: prompt presets** — 36 hardcoded English strings → `t('key')`
- [x] **Outline CSS** — hardcoded `#ef4444` → `var(--color-danger)` *(fixed in fix/ui-consistency)*
- [ ] **Brainstorm loading state** — no feedback during regeneration
- [x] **tokens.css** — define `--color-success-bg`, chip bg token *(fixed in fix/ui-consistency: added warning-bg, danger-bg, muted-bg)*
- [ ] **Schema.md** — add `sort_order` column
- [ ] **Accessibility** — `aria-pressed` on preset toggles, `aria-expanded` on outline header
- [ ] **reorderChapters E2E** — verify actual sort order persisted
- [ ] **Preset tests** — data integrity (unique IDs, valid categories)
- [ ] **Brainstorm parser tests** — en-dash separator, mixed formats
- [ ] **Outline save indicator** — no user feedback on outline auto-save
- [ ] **Redundant chapterId guard** — AISidebar handleBrainstorm

**Nits**
- [ ] parseBrainstorm regex inline comments
- [ ] Dev-mode logging for parseJson Zod failures

## Medium Priority

- [ ] **Split Product.md** - 1037 lines exceeds 150 line limit
- [ ] **Split Design.md** - 722 lines exceeds 150 line limit
- [ ] **Split Module documents** - All 10 modules exceed limits
- [ ] **Add release workflow** - GitHub Actions for releases
- [ ] **Create CONTRIBUTING.md** - Detailed contribution guide
- [ ] **Set up Codecov** - Code coverage reporting

## Low Priority

- [ ] **Add dependabot** - Automated dependency updates
- [ ] **Create project logo** - Inkstone/砚台 inspired design
- [ ] **Set up GitHub Pages** - Documentation site

---

## Completed

- [x] Initialize project structure (2026-02-05)
- [x] Create CLAUDE.md (2026-02-05)
- [x] Set up ADR system (2026-02-05)
- [x] Set up Milestone system (2026-02-05)
- [x] M1 Phase 1: Project Setup — monorepo, TS, ESLint, Vitest (2026-02-05)
- [x] M1 Phase 2: Interface Contract — all types and interfaces (2026-02-05)
- [x] Split Architecture documents 04, 05, 06 (2026-02-05)
- [x] Set up pre-commit hooks — husky + lint-staged (2026-02-05)
- [x] Fix ESLint v9 compatibility — typescript-eslint v8 (2026-02-05)
- [x] Create Regulation.md content (2026-02-05)
- [x] M1 Phase 4: Database Schema — Database class, MigrationRunner, 18 tables, FTS5 (2026-02-05)
- [x] M1 Phase 5: CLI Shell — init, serve, version, help commands + 9 tests (2026-02-05)
- [x] M1 Phase 6: Server + Web Shell — static serving, SPA fallback, AppShell, 4 pages (2026-02-05)
- [x] Update test mocks — MockEventBus AppEvent[], CreateCharacterInput literal unions (2026-02-05)
- [x] Codebase cleanup — ESM fixes, lint fixes, stale file removal (2026-02-05)
- [x] M1 merge to main — PR #1 merged (2026-02-06)
- [x] Draft M2–M5 milestone plans (2026-02-06)
