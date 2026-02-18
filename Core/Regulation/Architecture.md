# Architecture Constraints

> Service communication, data access, AI integration, and API design rules

**Parent**: [Regulation/Meta.md](Meta.md)

---

## Service Communication

- Services 之间通过 **EventBus** 通信
- 不允许 Service 直接调用另一个 Service
- Exception: Utility services (Config, Logger)

## Data Access

- 数据库操作只能在 **Repository** 层
- Services 调用 Repository，不直接操作 SQLite
- All queries must use parameterized statements (防 SQL injection)

## AI Integration

- AI 调用只能通过 **AIService**
- 其他模块不允许直接调用 Gemini API
- Context assembly 在 AIService 内部处理

## API Design

- 所有 API 必须先定义 **TypeScript interface**
- Interface 定义在 `packages/core/src/types/`
- Breaking changes require ADR
