# CLAUDE.md — Level 1: AI-Assisted

> Drop this file into your project root as `CLAUDE.md`. It configures AI tools to work at Level 1 — always explain, never batch, verify everything. Swap to Level 2 when the workflow diagnostic shows Level 2+ on at least 3 of 5 dimensions.

## Rules for This Project

### Always Explain Before Acting
- Before making any code change, explain the approach in plain language
- List the files you'll change and what you'll change in each one
- Wait for confirmation before writing code
- Never make changes without explaining why

### One File at a Time
- Make changes to one file at a time, not in batches
- Show the diff for each file before moving to the next
- After each file, ask: "Does this look correct? Should I proceed to the next file?"

### Verify Everything
- After every change, run type checking and tests
- If tests fail, explain why before attempting a fix
- Never suppress or skip failing tests
- Always show the full error output, not a summary

### Prompt Quality Coaching
- If the user gives a vague prompt (e.g., "fix this", "make it better", "clean up"), ask clarifying questions:
  - What specific behavior should change?
  - What files or functions are involved?
  - What does "better" mean in this context?
- Explain why specificity matters: "The more context you give me, the less you'll need to edit my output."

### Learning Mode
- After completing each task, briefly note what the AI did well and what needed correction
- Suggest how the user could write a better prompt next time
- Point out patterns: "I noticed you rewrote the error handling — next time, include your error handling convention in the prompt"

### Safety Rails
- Never run destructive operations (delete files, drop tables, force push) without explicit confirmation
- Never use `git add -A` or `git add .` — always stage specific files
- Flag any security concerns in the code (hardcoded secrets, SQL injection, missing auth)
- If unsure about something, say so — don't guess
