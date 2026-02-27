# 2026-02-26: M7 Visualizations — Code Review & Fixes

**Branch**: `feat/m7-track-b-visualizations`
**Milestone**: M7 Track B — Visualizations

---

## Completed

- Full code review of all 4 Visualization files (RelationshipMap, TimelineView, PacingView, index.tsx)
- **Component reuse**: replaced raw `<select>` → `Select`, raw `<button>` → `Button`, custom tab bar → `Tabs/TabPanel`
- **Null-check fix**: `!src.x` → `src.x == null` in RelationshipMap (allows 0 coordinate)
- **Timeline sort**: events now sort chronologically by ISO date, with fallback to insertion order for non-parseable strings (e.g. "百年前")
- **Arc filter restored**: indirect filtering via `arc.characterArcs` keys; empty arc returns 0 events (no silent skip)
- **Arc filter silent failure fix**: removed `if (arcCharIds.size > 0)` guard — honest 0-result behavior
- **formatDate fix**: ISO dates now format via `toLocaleDateString`; non-ISO strings pass through as-is
- **ErrorBoundary scope**: moved from wrapping all panels to wrapping each individual TabPanel
- **PacingView emotion interpolation**: replaced flat enum value with position-based interpolation (t = i/(n-1))
- **Zoom ctrl key**: `metaKey` → `metaKey || ctrlKey` — Windows/Linux users can now zoom
- **Arc chip buttons**: replaced raw `<button>` with `Button variant="ghost" size="sm"` in TimelineView
- **Outdated comment**: updated TimelineView doc comment to reflect actual arc filtering behavior
- **Dead code**: removed unused `stats.max` computation in PacingView
- **Simulation restart on resize**: refactored with `dimensionsRef`; resize only updates center force (no node reset); ResizeObserver guards against 0×0 reports
- **TabPanel unmounting**: changed from `return null` to `hidden` attribute — components stay mounted; d3 simulation survives tab switching
- **ARIA completeness**: `aria-controls` on tab buttons, `id="panel-{id}"` on panels, bidirectional link
- **Zoom buttons**: replaced 3 raw `<button>` zoom controls with `Button` component

## Modified Files

| File | Change |
|------|--------|
| `packages/web/src/components/ui/Tabs.tsx` | `aria-controls`, `id` on panels, `hidden` pattern (keep mounted) |
| `packages/web/src/pages/Visualizations/index.tsx` | `Tabs/TabPanel` component, ErrorBoundary per tab |
| `packages/web/src/pages/Visualizations/RelationshipMap.tsx` | Null checks, `Button` components, simulation/resize refactor, ctrlKey |
| `packages/web/src/pages/Visualizations/TimelineView.tsx` | Sort, arc filter, formatDate, arc chip buttons, comment |
| `packages/web/src/pages/Visualizations/PacingView.tsx` | Emotion interpolation, dead code removal |

## Stats

- Tests: **1270 passed (1 skipped)**
- Build: **clean**
