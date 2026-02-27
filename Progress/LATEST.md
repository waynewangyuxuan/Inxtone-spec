# LATEST

> Rolling 48h context recovery — last updated 2026-02-26

## Active Work

| Item | Status | Branch | Details |
|------|--------|--------|---------|
| M6: Smart Intake | Complete | `ms6` (merged) | All 4 phases done, 1270 tests passing |
| M7 Track B: Visualizations | In Progress | `feat/m7-track-b-visualizations` | Code review complete, ready to merge |

## Last 48h Summary

### 2026-02-26: Character factionId — Follow-up Fixes

- Fixed seed FK violation: seedRunner disables FK during exec (circular characters↔factions refs)
- Fixed can't-clear-faction: `factionId?: FactionId | null` + CharacterDetail passes `null`
- Fixed api.ts missing `factionId` in `CreateCharacterRequest`
- RelationshipMap factionColorMap: leader fallback + `character.factionId` primary
- CharacterDetail: added faction selector in character header
- **10 files modified** | **1270 tests, 0 failures** | build clean

### 2026-02-26: Character factionId — Bidirectional Faction Link

- Added `factionId?: FactionId` to `Character` type + `CreateCharacterInput`
- New migration 004: `faction_id TEXT REFERENCES factions(id)` on characters table
- Updated `CharacterRepository` (row type, create, update, mapRow)
- Updated demo seeds (zh + en) with faction assignments for all 6 characters
- Fixed `factionColorMap` in RelationshipMap: now colours all faction members, not just leaders
- **8 files modified** | **1270 tests, 0 failures** | build clean

### 2026-02-26: M7 Visualizations Code Review & Fixes

- Code review of RelationshipMap, TimelineView, PacingView, index.tsx + Tabs.tsx
- Replaced all raw `<button>` / `<select>` with design-system components (Button, Select, Tabs/TabPanel)
- Fixed timeline chronological sort, arc filter restore + silent-failure fix, formatDate formatting
- Fixed d3 simulation: resize no longer resets node positions (dimensionsRef + center force update)
- Fixed TabPanel: `hidden` attribute keeps components mounted across tab switches
- Fixed ARIA: `aria-controls` ↔ `id="panel-{id}"` bidirectional link
- ErrorBoundary scoped per tab; zoom now works with Ctrl (Windows/Linux)
- **5 files modified** | **1270 tests, 0 failures** | build clean

## Blockers

None currently.

## Next Up

- Merge `feat/m7-track-b-visualizations` → main
- Add "no results" empty state to TimelineView when filter returns 0 events (low priority)
- Plan remaining M7 tracks
