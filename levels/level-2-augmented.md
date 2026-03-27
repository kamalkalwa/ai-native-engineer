# CLAUDE.md — Level 2: AI-Augmented

> Drop this file into your project root as `CLAUDE.md`. It configures AI tools for Level 2 — codebase-aware, multi-file operations, test generation built into the workflow. Swap to Level 3 when the workflow diagnostic shows Level 3 on at least 3 of 5 dimensions.

## Rules for This Project

### Codebase-Aware by Default
- Before making changes, read the relevant files to understand existing patterns
- Follow the conventions already established in this codebase (naming, error handling, file structure)
- When suggesting new code, reference existing patterns: "Following the pattern in [file]..."
- If the codebase has inconsistent patterns, flag it and ask which pattern to follow

### Multi-File Operations
- For tasks touching 2+ files, propose a unified plan showing all changes before implementing
- Show dependencies between changes: what must happen before what
- Group related changes logically (model + service + route + test = one unit of work)
- After multi-file changes, run type checking to catch cross-file issues immediately

### Test Generation as Default
- Every PR-worthy change should include tests
- Generate behavior-based tests (test what it does, not how it does it)
- After generating tests, flag assertions that likely need human review:
  - Mock call count assertions
  - `toBeDefined()` or `toBeTruthy()` (too weak)
  - Tests that only verify the mock setup
- Expect ~30% of assertions to need rewriting — this is normal, not a failure

### Error Handling
- Follow the error handling pattern established in this codebase
- For new error paths, include: error type, useful message with context, and appropriate status code
- Never swallow errors silently — log or propagate

### PR Workflow
- When creating PRs, include: what changed, why, risk assessment, and rollback plan
- The rollback plan should be specific enough for an on-call engineer at 3am
- Stage specific files when committing — never `git add -A`

### Speed Over Perfection
- Don't ask for confirmation on every file — propose the full change set, then implement
- If a judgment call is needed, make it and flag it: "I chose X over Y because [reason]. Let me know if you'd prefer Y."
- Reserve confirmation requests for: destructive operations, architectural decisions, and ambiguous requirements
