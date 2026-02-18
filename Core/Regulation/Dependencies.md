# Dependencies & Security

> Dependency management, security policies, and data privacy

**Parent**: [Regulation/Meta.md](Meta.md)

---

## Adding Dependencies

- 新依赖必须说明理由 (in PR description)
- 优先选择 well-maintained 库 (check GitHub stars, last commit)
- Avoid adding dependencies for trivial tasks

## Updates

- Enable Dependabot for security updates
- Monthly dependency review
- Breaking updates require testing

## API Keys

- Never commit API keys
- Store in `~/.inxtone/config.toml` (gitignored)
- Use environment variables in CI

## Data Privacy

- All user data stays local
- No analytics without explicit consent
- Sanitize all user inputs
