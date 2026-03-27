---
name: pr
description: Full AI-assisted PR workflow — implementation review, test generation, self-review for security/performance/edge cases, PR description with risk assessment, commit and ship. Use when ready to open a PR.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash(git *), Bash(gh *), Bash(npx *), Bash(npm *)
---

# PR Workflow

Run the full AI-assisted PR workflow. Execute these steps in order.

## Step 1: Implementation Review

Review all changes made in this session (or on the current branch if no session context). Summarize:
- What changed (files and purpose)
- What the changes do together (the feature/fix as a whole)

Ask the user: "Does this summary match your intent? Anything missing before we proceed to tests?"

## Step 2: Test Generation

Generate tests for the changed code. Focus on:
1. NEW behavior introduced by the changes
2. EXISTING behavior that might be affected (regression tests)
3. Error conditions and edge cases

Rules:
- Test behavior, not implementation
- Mock external dependencies only
- Each test name completes "it should ___" with a product-visible behavior
- Assert on complete return values, not just existence
- Include at least 2 error-path tests per function changed

Run the tests after generation. Fix failures — if a test exposed a real bug, fix the implementation, not the test.

## Step 3: Self-Review

Review the complete diff for:

**Security:**
- User input passed to queries without sanitization
- Missing auth/authz checks on new endpoints
- Secrets in code, SSRF/XSS/injection vectors
- Insecure randomness, missing rate limiting

**Performance:**
- N+1 queries, missing indexes, missing pagination
- Large objects in memory, sync operations that should be async

**Edge cases:**
- Empty/null/undefined input, very large input
- Concurrent requests, external service failures
- Unicode, special characters, extremely long strings

**Logic:**
- Off-by-one errors, race conditions, missing await

For each issue found, output: `| Severity | File:Line | Issue | Fix |`

Fix CRITICAL and HIGH issues. Note MEDIUM issues for the PR description.

## Step 4: PR Description

Generate:

```
## What Changed
[Bullet list — each starting with a verb: Added, Updated, Removed, Fixed]

## Why
[1-2 sentences — the business reason, not the technical reason]

## Risk Assessment
| Risk | Likelihood | Impact | Mitigation |

## Rollback Plan
- Feature flag: [Yes/No — which flag]
- Database migrations: [Yes/No — reversible?]
- Breaking API changes: [Yes/No — who's affected]
- Steps: [numbered rollback actions]

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing: [specific scenarios]
- [ ] Edge cases: [specific cases tested]
```

Ask the user to review — especially the risk assessment and rollback plan. AI tends to understate risk and write generic rollback steps.

## Step 5: Ship

Stage specific files (never `git add -A`), commit with a clear message, push, and create the PR.
