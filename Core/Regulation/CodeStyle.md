# Code Style

> TypeScript conventions, naming, and formatting rules

**Parent**: [Regulation/Meta.md](Meta.md)

---

## TypeScript

- **Strict mode** enabled (`"strict": true`)
- 禁止 `any`，必须显式类型
- Prefer `interface` over `type` for object shapes
- Use `readonly` where applicable

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Variables/Functions | camelCase | `getUserById`, `isValid` |
| Classes/Types/Interfaces | PascalCase | `UserService`, `Character` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRIES`, `API_BASE_URL` |
| Files | kebab-case | `user-service.ts`, `story-bible.ts` |
| Folders | kebab-case | `services/`, `utils/` |

## Formatting

- **ESLint** + **Prettier** enforced
- Import ordering: external → internal → relative
- Max line length: 100 characters
- 2 spaces indentation
