# Workflow: Full AI-Assisted PR Workflow

**Time:** Reduces PR cycle by ~50%
**Tools:** Claude Code (CLI) + Cursor Composer
**Level:** 2-3 (AI-Augmented to AI-Native)

---

## When to Use This

Every PR. This isn't a special workflow — it's the default workflow for AI-native engineers. The steps below replace the traditional cycle of: code manually, write tests manually, write PR description manually, wait for review, fix nits, repeat.

---

## The Full Workflow

### Step 1: Generate Implementation (15-45 min depending on scope)

Start with Claude Code. Describe the feature or change:

```
Read the codebase context for [relevant area]. Here's what I need to build:

## Task
[paste ticket/issue description or write your own]

## Constraints
- Must work with existing [authentication/database/API] patterns in this codebase
- Must not break [specific existing functionality]
- [Any performance requirements, backward compatibility needs, etc.]

## Acceptance Criteria
- [Specific, testable criteria — not vague descriptions]
- [Include error cases: "When X fails, the user should see Y"]
- [Include edge cases you already know about]

Propose an implementation approach first. List the files you'll change
and what you'll change in each one. Do not write code yet.
```

**Review the approach.** This is where your engineering judgment matters most. Challenge it:

```
What are the edge cases in this approach?
What happens if [specific failure scenario]?
Is there a simpler way to achieve [specific part]?
```

Once the approach is solid:

```
Implement the approach. Show me all file changes.
```

**Review the implementation.** Look for:
- Does it follow existing codebase patterns?
- Are there hardcoded values that should be configurable?
- Is error handling complete?
- Are there missing null checks or type guards?

Iterate on specific files if needed:

```
In src/services/orderService.ts, the createOrder function doesn't handle
the case where inventory is 0. Add a check that returns an appropriate
error before attempting the charge.
```

---

### Step 2: Generate Tests (15-30 min)

Use the test generation workflow from [test-generation.md](./test-generation.md). The prompt:

```
Generate tests for the changes I just made. Focus on:

1. The NEW behavior introduced by this change
2. EXISTING behavior that might be affected (regression tests)
3. Error conditions and edge cases

Rules:
- Test behavior, not implementation
- Mock external dependencies only
- Each test name completes "it should ___" with a product-visible behavior
- Assert on complete return values, not just existence
- Include at least 2 error-path tests per function changed

Testing framework: [jest/vitest/pytest]
```

Apply the 30% rewrite rule: review every assertion, rewrite implementation-testing assertions to behavior-testing assertions.

Run the tests:

```bash
npm test
```

Fix any failures. If the tests exposed a bug in the implementation, fix the implementation — don't change the test to match buggy behavior.

---

### Step 3: Generate PR Description (5 min)

```
Generate a PR description for the changes in this session.

Format:

## What Changed
[Bullet list of changes, each starting with a verb: Added, Updated, Removed, Fixed]

## Why
[1-2 sentences on the business reason, not the technical reason]

## How It Works
[Brief technical explanation for reviewers — just enough to understand the approach without reading every line]

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| [specific thing that could go wrong] | Low/Medium/High | [what would happen] | [how to detect and fix] |

## Rollback Plan
[Exact steps to rollback if this change causes issues in production]
- Is this behind a feature flag? [Yes/No — if yes, which flag]
- Database migrations? [Yes/No — if yes, are they reversible?]
- Breaking API changes? [Yes/No — if yes, who's affected?]
- Steps: [numbered list of rollback actions]

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed for: [list specific scenarios tested]
- [ ] Edge cases tested: [list specific edge cases]

## Screenshots/Recordings
[If UI changes, include before/after]
```

**Review the PR description.** AI tends to:
- Understate risk (add risks it missed)
- Be vague on rollback (make it specific enough that an on-call engineer at 3am can follow it)
- Skip the "Why" or make it too technical (rewrite for your audience)

---

### Step 4: Self-Review for Security, Performance, and Edge Cases (10 min)

Before requesting human review, run the AI self-review. This catches a surprising amount of issues:

```
Review the complete diff of changes in this session. Look specifically for:

1. **Security vulnerabilities:**
   - User input passed to SQL/NoSQL queries without sanitization
   - Missing authentication or authorization checks on new endpoints
   - Secrets or credentials in code
   - SSRF, XSS, or injection vectors
   - Insecure randomness (Math.random for tokens)
   - Missing rate limiting on new endpoints

2. **Performance issues:**
   - N+1 queries (loop that makes a DB call per iteration)
   - Missing database indexes for new query patterns
   - Large objects held in memory unnecessarily
   - Missing pagination on list endpoints
   - Synchronous operations that should be async
   - Missing caching for expensive, repeated operations

3. **Edge cases:**
   - What happens with empty input? Null? Undefined?
   - What happens with very large input? (1M items in a list, 10MB payload)
   - What happens with concurrent requests to the same resource?
   - What happens if an external service (DB, API, cache) is down?
   - What happens with unicode, special characters, or extremely long strings?

4. **Logic errors:**
   - Off-by-one errors in pagination, loops, or array slicing
   - Race conditions in async code
   - Missing await on async calls
   - Error states that silently succeed

For each issue found, output:
| Severity | File:Line | Issue | Fix |
```

**Act on the findings.** Fix CRITICAL and HIGH issues before opening the PR. Note MEDIUM issues in the PR description as known limitations or follow-up tasks.

---

### Step 5: Open the PR

```bash
# Stage all changes
git add src/path/to/changed-files   # add specific files, never use -A

# Commit with a clear message
git commit -m "feat: add order cancellation with inventory restoration

- Add cancelOrder endpoint with idempotency check
- Restore inventory on successful cancellation
- Send cancellation confirmation email
- Add rate limiting (10 cancellations per hour per user)"

# Push and create PR
git push -u origin feature/order-cancellation
gh pr create --title "Add order cancellation with inventory restoration" \
  --body "$(cat pr-description.md)"
```

Or use Claude Code's built-in PR creation:

```
Create a PR for this branch. Use the PR description we generated.
Target branch: main
```

---

## Time Breakdown: Traditional vs. AI-Assisted

| Step | Traditional | AI-Assisted | Notes |
|------|------------|-------------|-------|
| Understand codebase context | 30-60 min | 5-10 min | AI reads the repo instantly |
| Plan implementation | 15-30 min | 10-15 min | You still make the design decisions |
| Write implementation | 60-180 min | 15-45 min | AI drafts, you review and shape |
| Write tests | 60-120 min | 15-30 min | AI generates, you rewrite 30% |
| Write PR description | 15-30 min | 5 min | AI drafts, you adjust risk/rollback |
| Self-review | 15-30 min | 10 min | AI catches mechanical issues, you catch design issues |
| **Total** | **3.5-7.5 hours** | **1-2 hours** | **~50-70% reduction** |

The time savings come from parallelizing with AI, not from cutting corners. Every step still involves human review and judgment. The AI handles the generation; you handle the verification.

---

## Handling PR Review Feedback

When reviewers leave comments, use AI to address them efficiently:

```
Here are the review comments on my PR:

[paste review comments]

For each comment:
1. Explain whether the reviewer's concern is valid
2. If valid, show me the fix
3. If it's a misunderstanding, draft a response explaining why
   the current approach is correct (cite specific code)
```

**Do not blindly apply AI-generated fixes to review comments.** Read the reviewer's concern, understand it, then decide if the AI's fix addresses the actual concern or just the surface-level comment.

---

## Common Pitfalls

1. **Don't skip the approach review (Step 1).** Going straight to "write the code" produces code that works but doesn't fit your system. The approach review is where you catch architectural mismatches before they're embedded in 500 lines of code.

2. **Don't use the AI-generated PR description without editing.** AI descriptions are technically accurate but miss team context. Add: why this approach was chosen over alternatives, what you're unsure about, what you'd like reviewers to focus on.

3. **Don't skip the self-review (Step 4).** It takes 10 minutes and consistently catches issues that would otherwise come up in review (costing both you and the reviewer time). The ROI is massive.

4. **Don't let AI handle the rollback plan.** AI writes generic rollback plans ("revert the commit"). Write a specific one: what to check, what order to do things, who to notify, what data might be affected.

5. **Don't use this workflow for trivial changes.** A one-line config change doesn't need AI-generated tests, risk assessment, and rollback plan. Use this for feature work, refactors, and changes that touch multiple files.
