# /end-session

> Session wrap-up protocol — record progress, update context, check compliance

## Trigger

Run at the end of each AI coding session, before final commit/push.

## Steps

1. **Summarize changes**
   ```bash
   git diff --stat HEAD
   ```
   Categorize: new files, modified files, deleted files.

2. **Generate Progress file**
   - Create `spec/Progress/{YYYY-MM-DD}_{slug}.md`
   - Template:
     ```markdown
     # {date}: {summary}
     **Branch**: {current branch}
     **Milestone**: {current milestone}
     ---
     ## Completed
     {bullet list of work done}
     ## New Files
     {table if any}
     ## Modified Files
     {table if any}
     ## Stats
     - Tests: {count}
     - Build: {status}
     ```

3. **Update LATEST.md**
   - Update Active Work table with current state
   - Update Last 48h Summary (keep only entries from last 48h)
   - Update Blockers section
   - Update Next Up section

4. **Compliance checks**
   - [ ] All new spec files <= 150 lines
   - [ ] All new folders have Meta.md
   - [ ] Regulation changes are append-only (no overwrites)
   - [ ] Progress file follows naming convention

5. **Push spec changes** (if submodule)
   ```bash
   cd spec/ && git add -A && git commit -m "progress: {slug}" && git push
   cd .. && git add spec && git commit -m "docs: update spec submodule"
   ```

## Notes

- If no spec/ submodule, write Progress files to `Meta/Progress/` directly
- If compliance checks fail, report issues but don't block — let user decide
- The slug should be 2-3 words describing the session (e.g., `M7_quality_rules`)
