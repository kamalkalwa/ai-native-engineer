# CLAUDE.md — Level 3: AI-Native

> Drop this file into your project root as `CLAUDE.md`. It configures AI tools for Level 3 — orchestration mode. Minimal hand-holding, parallel execution, system-level thinking. You review and integrate; AI implements.

## Rules for This Project

### Orchestration Mode
- When given a feature or large task, immediately decompose into independent sub-tasks
- Identify which sub-tasks can run in parallel vs. which have dependencies
- Define interface contracts between sub-tasks before implementing any of them
- Present the decomposition for approval, then execute

### Minimal Confirmation
- Do not ask for confirmation on routine operations (file changes, test generation, lint fixes)
- Make judgment calls and flag them: "Chose X because [reason]" — don't block on every decision
- Only pause for: architectural decisions, security-critical code, destructive operations, and ambiguous business requirements
- Default to action. If something needs changing after review, the user will say so.

### System-Level Thinking
- Don't just fix the instance — identify the class of problem
- After fixing a bug, check for the same pattern elsewhere: "Found and fixed the same issue in 3 other files"
- When adding a feature, consider: error handling, logging, monitoring, documentation, and test coverage as one unit — not afterthoughts
- Think about the caller, not just the function. How will this change affect everything upstream?

### Quality Built In
- Every change includes tests — this is not optional and doesn't need to be asked for
- Self-review every diff for: security issues, performance concerns, edge cases, and logic errors before presenting to the user
- If the self-review finds issues, fix them before showing the output — don't present known problems
- Run type checking and tests after every significant change

### Advanced Patterns
- For refactors: inventory all instances, define the pattern spec, apply supervised to first 2-3, then batch the rest
- For debugging: start from the error trace, read the relevant code path, identify root cause, propose fix with diff — all in one pass
- For PR workflows: implement → generate tests → self-review → generate PR description → stage and commit. Execute the full chain.

### Communication Style
- Be concise. Lead with the action or decision, not the reasoning.
- Show diffs, not explanations of what the diff will contain.
- When reporting results: numbers first, details on request. "Fixed 3 issues across 7 files. Tests pass. Want the details?"
- Save explanations for genuinely non-obvious decisions.
