# LATEST

> Rolling 48h context recovery — last updated 2026-07-01

## Active Work

| Item | Status | Branch | Details |
|------|--------|--------|---------|
| M7 Track B: Visualizations | **Merged** | PR #21 → main | 4-month-old review finally landed |
| Gemini deprecation + toolchain fixes | **Merged** | PR #22 → main | model IDs, pnpm 10, critical CVEs, doc drift |
| Fastify 5 migration | Open PR | PR #23 (`fix/fastify-5-migration`) | awaiting review |
| useAutoSave abort fix + web test infra | Open PR | PR #24 (`fix/use-autosave-race`) | awaiting review |
| Product redirection | Proposed | — | ADR-0007/0008/0009 |

## Last 48h Summary

### 2026-07-01: Return Session — Audit, Fixes, Redirection

**Health audit (3 parallel reports)** after 4-month pause:
- *Broken in prod*: `gemini-2.0-flash` shut down by Google 2026-06-01 while hardcoded in `verifyApiKey()` + init template → API-key verification failed for all users. pnpm 9 toolchain broken on machine. 54 security vulns (2 critical incl. runtime protobufjs RCE chain).
- *Fixed via PR #22 (merged)*: models → `gemini-3.5-flash`, `packageManager` → pnpm 10.8.1 + `onlyBuiltDependencies`, `@google/genai` + mammoth upgrades, README/CLAUDE.md drift. Vulns 54 → 45.
- *Fixed via PR #23 (open)*: Fastify 4 (EOL) → 5; removed unused `@fastify/websocket`; vulns 45 → 34, server-path clean.
- *Fixed via PR #24 (open)*: useAutoSave — prior fix didn't abort in-flight PUTs at network layer; now real AbortSignal. + jsdom/@testing-library infra + 6 unit tests (1276 passing total).

**Redirection (Wayne's direction + 5 research reports → 3 ADRs, all Proposed)**:
- **ADR-0007**: Inxtone = *opinionated knowledge base for fiction writing*; dual-host: Phase 1 Claude Code plugin (MCP server + skill pack + `inxtone view` codex), Phase 2 own conversational CLI; cloud runtime rejected. MCP tools = CRUD + opinionated queries (`bible_whats_due`, `bible_check_wayne_r1`).
- **ADR-0008**: Design language 「纸墨朱砂 × 铅笔手绘」— paper ground, ink-dilution hierarchy, cinnabar single accent, Excalidraw-style hand-drawn strokes; CJK typography fixes (fonts actually bundled, ≥400 weight, no italics, lang=zh-CN); includes 1-day triage list.
- **ADR-0009**: Context engineering — 3 tiers (resident-cached / retrieved / working); taxonomy-as-index (rules > FTS5 > vectors); methodology = one registry, three forms (systemInstruction / QualityService / few-shot). ADR-0002 recommended → Accepted.

Spec drift fixed: ADR-0001 sqlite-vss note, 03_ai_service + M3 model-name notes, Appendix Gemini-risk Low→High.

## Blockers

None. Awaiting Wayne's review of PR #23/#24 and sign-off on ADR-0007/0008/0009.

## Next Up

- Review & merge PR #23 (Fastify 5), PR #24 (useAutoSave)
- ADR sign-off: 0007 (positioning), 0008 (design), 0009 (context) — then 0002 → Accepted
- UI triage list (~1 day, ships before redesign): bundle CJK fonts, weight 400, 15/16px, no CJK italics, lang=zh-CN, API-key modal skip persistence, merge duplicate tokens
- Plan M7 Track A (QualityService — now doubles as the shared rule registry of ADR-0009)
- `@inxtone/mcp` Phase 1 prototype (ADR-0007)
