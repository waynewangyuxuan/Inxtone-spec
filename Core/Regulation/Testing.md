# Testing

> Coverage targets, TDD workflow, contract tests, and git worktree

**Parent**: [Regulation/Meta.md](Meta.md)

---

## Coverage Targets

| Layer | Target |
|-------|--------|
| Services | >=80% |
| Utils | >=90% |
| Components | >=70% |

## Testing Rules

- **New feature** → must have unit tests
- **Bug fix** → write failing test first, then fix
- **Refactor** → ensure existing tests pass
- Use `describe` / `it` naming: `describe('UserService')`, `it('should return user by id')`

## Test-Oriented Development

### Why Test-First for Parallel Development

When multiple branches develop in parallel (via git worktree), **tests are the contract**:

- Interfaces define **what** the API looks like
- Tests define **how** it should behave
- Implementation can vary, but tests must pass

### Workflow: Phase Start Ritual

**Before writing any implementation code:**

1. **Run `/test-plan` skill** to generate test plan
2. **Define interfaces** in `packages/core/src/types/`
3. **Write test stubs** (failing tests OK at this stage)
4. **Commit**: `test(scope): add test stubs for [feature]`
5. **Then implement** until tests pass

```
+----------------------------------------------------------+
|  Phase Start                                              |
+----------------------------------------------------------+
|  1. /test-plan -> generates test cases                    |
|  2. Write interfaces (types/*.ts)                         |
|  3. Write test stubs (*.test.ts)                          |
|  4. Commit test stubs                                     |
|  5. Implement until green                                 |
|  6. Commit implementation                                 |
+----------------------------------------------------------+
```

## Test Categories

| Category | Location | Purpose |
|----------|----------|---------|
| **Unit** | `*.test.ts` next to source | Single function/class |
| **Integration** | `__tests__/integration/` | Service interactions |
| **E2E** | `__tests__/e2e/` | Full user flows |
| **Contract** | `__tests__/contracts/` | API interface compliance |

## Contract Tests

Contract tests ensure parallel branches stay compatible:

```typescript
// __tests__/contracts/character-service.contract.ts
describe('CharacterService Contract', () => {
  it('should implement ICharacterService interface', () => {
    const service: ICharacterService = new CharacterService(db);
    expect(service).toBeDefined();
  });

  it('should return Character type from getById', async () => {
    const result = await service.getById('test-id');
    expect(result).toMatchSchema(CharacterSchema);
  });
});
```

## Git Worktree Workflow

```bash
# Setup parallel development
git worktree add ../inxtone-bible  feat/m2-story-bible
git worktree add ../inxtone-write  feat/m3-writing
git worktree add ../inxtone-ai     feat/m4-ai

# Each worktree runs tests independently
cd ../inxtone-bible && pnpm test

# Before merge: run ALL tests
git checkout main
pnpm test:all
```

## Merge Criteria

Branch can only merge to main when:

- [ ] All unit tests pass
- [ ] All contract tests pass
- [ ] No type errors (`pnpm typecheck`)
- [ ] Coverage maintained (>=80% for services)
