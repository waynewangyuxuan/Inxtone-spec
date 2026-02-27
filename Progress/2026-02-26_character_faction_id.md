# 2026-02-26: Character factionId — Bidirectional Faction Link

**Branch**: `feat/m7-track-b-visualizations`
**Milestone**: M7 Track B — Visualizations

---

## Problem

`RelationshipMap` "By Faction" colour mode only coloured faction leaders because
`Character` had no `factionId` field. The only link was `faction.leaderId`
(faction → character), so non-leader members always fell back to role colour.

## Completed

- **`entities.ts`**: Added `factionId?: FactionId` to `Character` interface
- **`services.ts`**: Added `factionId?: FactionId` to `CreateCharacterInput` (automatically available on `UpdateCharacterInput` via `Partial<>`)
- **`004_character_faction_id.ts`**: New migration adds `faction_id TEXT REFERENCES factions(id) ON DELETE SET NULL` + index to `characters` table
- **`migrations/index.ts`**: Registered migration004
- **`CharacterRepository.ts`**: Updated `CharacterRow`, `create`, `update`, and `mapRow` to handle `faction_id`
- **`sql-zh.ts`** (demo seed): Added `faction_id` assignments:
  - C-001 沈书 → F-001 七院联盟 (from book academy)
  - C-002 落棠 → F-001 七院联盟 (剑宗 is one of the seven academies)
  - C-003 墨痴 → F-002 逐墨众 (leader)
  - C-004 阿鹿 → NULL (mountain hunter, unaffiliated)
  - C-005 钟离白 → F-003 守章人 (leader)
  - C-006 句读先生 → NULL (mysterious wanderer)
- **`sql-en.ts`** (English seed): Same column added with corresponding assignments
- **`RelationshipMap.tsx`**: `factionColorMap` now builds `faction id → color` first, then maps characters via `c.factionId` — all faction members are coloured, not just leaders

## Modified Files

| File | Change |
|------|--------|
| `packages/core/src/types/entities.ts` | Add `factionId?: FactionId` to `Character` |
| `packages/core/src/types/services.ts` | Add `factionId?: FactionId` to `CreateCharacterInput` |
| `packages/core/src/db/migrations/004_character_faction_id.ts` | **New** migration |
| `packages/core/src/db/migrations/index.ts` | Register migration004 |
| `packages/core/src/db/repositories/CharacterRepository.ts` | Row type + CRUD + mapRow |
| `packages/core/src/db/seeds/sql-zh.ts` | Add `faction_id` column + values |
| `packages/core/src/db/seeds/sql-en.ts` | Add `faction_id` column + values |
| `packages/web/src/pages/Visualizations/RelationshipMap.tsx` | Fix `factionColorMap` |

## Stats

- Tests: **1270 passed (1 skipped)**
- Build: **clean**
