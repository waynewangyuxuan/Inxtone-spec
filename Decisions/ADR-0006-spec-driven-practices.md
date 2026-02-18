# ADR-0006: Adopt Spec-Driven Collaboration Practices

- **Status**: Accepted
- **Date**: 2026-02-18
- **Deciders**: Wayne

## Context

Inxtone's documentation has grown beyond the 150-line convention:
- `Regulation.md` at 286 lines
- `Progress.md` at 1259 lines

The spec-driven-collaboration knowledge module provides a framework for structuring documentation, managing sessions, and scaling to multi-person AI Coder teams.

## Decision

Adopt the **structural subset** of spec-driven collaboration practices:

### Adopted
1. **Regulation.md split** — 286-line monolith -> 7 focused files in `Regulation/` folder
2. **Progress/ directory** — 1259-line monolith -> individual session files + `LATEST.md` (rolling 48h)
3. **Skills/ directory** — `/start-session` and `/end-session` workflow skills
4. **Spec submodule** — `Meta/` content extracted to separate repo, linked at `spec/`
5. **Session protocol** — Standardized context loading and progress recording
6. **Milestone parallel tracks** — M7/M8 restructured into 3 parallel tracks for worktree development

### Deferred
- GitHub Rulesets (two-level permissions) — solo developer, revisit when team grows
- Issue templates with Spec Context field — can add later
- PR templates (Code PR / Spec PR) — can add later
- CI enforcement (150-line check, Meta.md check) — manual for now

## Alternatives Considered

1. **Full adoption** — All 10 sections of the knowledge module. Rejected: too much infrastructure overhead for solo developer.
2. **No adoption** — Keep current structure. Rejected: Regulation.md and Progress.md already violating conventions.
3. **Partial structural only** — Split files but no submodule or skills. Rejected: submodule enables proper separation and skills standardize workflow.

## Consequences

### Positive
- All documentation files comply with 150-line convention
- `LATEST.md` provides fast context recovery between sessions
- Parallel milestone tracks enable worktree-based development
- Spec submodule separates documentation and code commit rhythms

### Negative
- Submodule adds maintenance overhead (sync, hooks)
- More files to navigate (mitigated by Meta.md routing)
- Skills need to be kept in sync with actual workflow

## Knowledge Source

Prism output/spec-driven-collaboration
