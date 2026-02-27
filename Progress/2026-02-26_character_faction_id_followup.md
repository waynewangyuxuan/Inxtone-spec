# 2026-02-26: Character factionId — Follow-up Fixes

**Branch**: `feat/m7-track-b-visualizations`
**Milestone**: M7 Track B — Visualizations

---

## Completed

Code review of the `factionId` data model change revealed 3 issues; all fixed:

### Bug fixes

- **CRITICAL — Seed FK violation**: `characters` (with `faction_id`) were inserted before
  `factions` in the seed SQL, violating `PRAGMA foreign_keys = ON`.
  `characters.faction_id ↔ factions.leader_id` is a circular reference that cannot be
  resolved in single-pass insert order. Fixed by temporarily disabling FK enforcement in
  `seedRunner.ts` around `db.exec(sql)`.

- **Can't clear faction**: `CharacterDetail` passed `undefined` when user selected
  "No Faction", but `CharacterRepository.update()` only runs the SQL when
  `input.factionId !== undefined` → faction was impossible to unset.
  Fixed by: (1) widening `factionId?: FactionId | null` in `CreateCharacterInput`
  (and thus `UpdateCharacterInput`), (2) `CharacterDetail` now passes `null` for empty selection.

- **api.ts type inconsistency**: `CreateCharacterRequest` / `UpdateCharacterRequest` in
  `api.ts` were missing `factionId`, making the API contract out of sync with `services.ts`.
  Added `factionId?: FactionId` to `CreateCharacterRequest` and imported `FactionId`.

### Additional improvements (same session)

- **RelationshipMap factionColorMap**: Added leader fallback so characters whose
  `factionId` hasn't been set yet still receive a faction colour if they are listed as
  `faction.leaderId`. New strategy: leader fallback first, then `character.factionId`
  overwrites (more accurate). Fixes "By Faction" showing role colours for all nodes.

- **CharacterDetail faction selector**: Added inline `EditableField` (select) to the
  character header so users can assign/clear faction membership directly from the Story
  Bible. Only rendered when the story has at least one faction.

## New Files

| File | Purpose |
|------|---------|
| `packages/core/src/db/migrations/004_character_faction_id.ts` | Migration: adds `faction_id` column + index to `characters` |

## Modified Files

| File | Change |
|------|--------|
| `packages/core/src/types/entities.ts` | `Character` + `factionId?: FactionId` |
| `packages/core/src/types/services.ts` | `CreateCharacterInput.factionId?: FactionId \| null` |
| `packages/core/src/types/api.ts` | `CreateCharacterRequest` + `factionId?`; import `FactionId` |
| `packages/core/src/db/migrations/index.ts` | Register migration004 |
| `packages/core/src/db/repositories/CharacterRepository.ts` | Row type, create, update, mapRow |
| `packages/core/src/db/seeds/seedRunner.ts` | FK OFF/ON around `db.exec(sql)` |
| `packages/core/src/db/seeds/sql-zh.ts` | Add `faction_id` column + values |
| `packages/core/src/db/seeds/sql-en.ts` | Add `faction_id` column + values |
| `packages/web/src/pages/Visualizations/RelationshipMap.tsx` | `factionColorMap` with leader fallback + `factionId` primary |
| `packages/web/src/pages/StoryBible/CharacterDetail.tsx` | Faction selector in character header |

## Stats

- Tests: **1270 passed (1 skipped)**
- Build: **clean**
