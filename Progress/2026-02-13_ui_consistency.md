# 2026-02-13: UI Consistency & Tech Debt Sweep

**Branch**: `fix/ui-consistency`

---

## Completed

**P0: Critical Fixes**
- Removed debug `console.log` from CharacterForm.tsx
- Replaced all hardcoded colors with CSS tokens across TSX and CSS modules
- Added missing design tokens to `tokens.css`: `--color-warning-text`, `--color-warning-bg`, `--color-danger-bg`, `--color-muted-bg`
- Unified page header naming: `.subtitle` as primary class

**P1: Structural Improvements**
- Created reusable `LoadingSpinner` component — replaced 16 ad-hoc spinner implementations
- Deduplicated `@keyframes spin` into `global.css`
- Migrated all inline styles in Export.tsx (14) and Settings.tsx (6) to CSS modules
- Added form utility classes to `Page.module.css`
- Replaced blocking `alert()` calls with toast notification system (Zustand + NotificationToast)
- Standardized Unicode chevron characters across all toggles

**P2: Polish**
- Added `chevronDown`, `chevronRight`, `close` icons to `Icon.tsx`
- Tokenized Badge, Button, Input, HookTracker, Dashboard CSS modules

## Stats
- **5 new files, 25+ modified**
