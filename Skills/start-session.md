# /start-session

> Session startup protocol — load context, understand current state, output constraints

## Trigger

Run at the beginning of each AI coding session.

## Steps

1. **Pull spec** (if submodule exists)
   ```bash
   git submodule update --remote spec/
   ```

2. **Read recent context**
   - `spec/Progress/LATEST.md` — rolling 48h summary, active work, blockers
   - `spec/Todo.md` — current task backlog

3. **Load issue context** (if working on a specific issue)
   - Read the GitHub issue body
   - Identify Spec Context paths listed in the issue
   - Load those spec files

4. **Read constraints**
   - `spec/Core/Regulation/Meta.md` — scan for relevant regulations
   - Load specific regulation files based on the task type

5. **Output summary**
   ```
   Session Ready
   - Last activity: {from LATEST.md}
   - Current task: {from Todo.md or issue}
   - Constraints loaded: {regulation files read}
   - Blockers: {from LATEST.md or "none"}
   ```

## Notes

- If no spec/ submodule exists, read from `Meta/` directly
- If LATEST.md doesn't exist, read the most recent Progress session file
- This skill is informational — it does not modify any files
