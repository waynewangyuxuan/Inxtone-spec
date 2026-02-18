# Git Workflow

> Branching, commit conventions, and PR process

**Parent**: [Regulation/Meta.md](Meta.md)

---

## Branching

| Prefix | Usage |
|--------|-------|
| `feat/` | New features |
| `fix/` | Bug fixes |
| `docs/` | Documentation only |
| `refactor/` | Code refactoring |
| `test/` | Test additions |
| `chore/` | Maintenance tasks |

Example: `feat/add-character-editor`, `fix/export-unicode-issue`

## Commits

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): description

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`

## Pull Requests

- 必须有清晰描述
- 关联 Issue 或 Milestone task
- 不允许直接 push to `main`
- Squash merge preferred
