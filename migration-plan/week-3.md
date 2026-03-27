# Week 3: Multi-File Operations and Harness Prompts

**Goal:** Execute a codebase-wide refactor using AI. Move from single-task AI usage to system-level AI operations. By end of week, you should have completed at least one refactor that would have taken days and took hours.

**Level transition:** Solidifying Level 2 (AI-Augmented), preparing for Level 3

> **Reference:** This week applies the [Multi-File Refactor Workflow](../workflows/multi-file-refactor.md) step-by-step. Read that workflow first — it covers the approach in detail. This week's plan structures it into daily execution.

---

## Day 1 (Monday): Identify Your Refactor Target

### The Exercise

You need one real refactor to execute this week. Pick from this list based on what your codebase actually needs:

| Refactor Type | Good If... | Scope |
|---------------|-----------|-------|
| Error handling migration (try/catch to Result type) | You have inconsistent error handling | 10-30 files |
| Logging standardization | Logs use different formats/levels across services | 10-50 files |
| API response format unification | Endpoints return data in different shapes | 5-15 files |
| Dependency injection refactor | Services instantiate their own dependencies | 10-20 files |
| Import path migration (relative to absolute) | You have `../../../` paths everywhere | 20-100 files |
| Configuration centralization | Config is scattered across files | 10-30 files |

### Exercise: Inventory the Current State

Before refactoring, understand the scope. Run this in Claude Code:

```
Find every instance of [current pattern] in this codebase.

For each instance, output:
1. File path and line numbers
2. The exact current code
3. What this specific instance does (context for the replacement)
4. Complications: does this instance have special behavior, side effects,
   or edge cases that make it different from the standard pattern?

Output as a numbered list. Be exhaustive — I need a complete inventory
before I start changing anything.
```

### Exercise: Write the Target Pattern Specification

This is the most important document you'll write this week. It defines exactly what the new pattern looks like. Include:

1. **The type/interface definitions** (if applicable)
2. **Before/After examples** — at least 2, showing different variants
3. **Transformation rules** — mechanical rules the AI can follow
4. **Exceptions** — files or patterns to NOT change
5. **Import changes** — what to add/remove

Write this in a file (e.g., `REFACTOR_SPEC.md` in your repo) so you can reference it in prompts.

### Success Criteria for Day 1
- [ ] Refactor target is chosen based on real codebase need
- [ ] Complete inventory of all instances (you know the number and locations)
- [ ] Target pattern specification is written with before/after examples and transformation rules

---

## Day 2 (Tuesday): Execute the Refactor (First Half)

### Morning: First 3 Files (Supervised)

Apply the refactor to the first 3 files one at a time, reviewing each carefully:

```
Apply the refactor pattern from [REFACTOR_SPEC.md or paste the spec]
to [file path].

Show me the before and after for each change in this file.
Flag anything that doesn't fit the standard pattern.
```

Review each transformation. Look for:
- Does the AI follow your spec exactly?
- Are there edge cases it handled incorrectly?
- Does it preserve existing behavior?

If the AI consistently deviates from your spec, correct the spec or the prompt before proceeding.

### Afternoon: Batch Mode (Files 4-N/2)

Once you're confident the pattern is applied correctly:

```
Apply the refactor pattern to these files:
[list remaining files for today — approximately half of the total]

For each file:
1. Apply the standard transformation
2. Update imports
3. Update callers in this file if they reference changed functions
4. Flag any instance that required a non-standard transformation

Show all changes grouped by file.
```

### Exercise: Run Type Checking After Each Batch

After each batch of changes:

```bash
npx tsc --noEmit 2>&1 | head -50
```

If there are type errors, feed them back:

```
Here are the TypeScript errors after the refactor batch:
[paste errors]

Fix each one. Show me the fixes grouped by file.
```

### Success Criteria for Day 2
- [ ] First 3 files reviewed individually — pattern is correct
- [ ] Approximately half of all files are refactored
- [ ] Type checking passes (or errors are identified and tracked)

---

## Day 3 (Wednesday): Complete the Refactor

### Morning: Remaining Files

Apply the refactor to all remaining files:

```
Apply the refactor pattern to the remaining files:
[list them]

Same rules as yesterday. Flag non-standard transformations.
After all files are done, run through the complete list and verify
no instances were missed.
```

### Afternoon: Update Callers

This is the step most people forget. Changing a function's signature or return type breaks its callers.

```
The refactor changed the following function signatures:
[list changed functions or say "all functions listed in the inventory"]

Find every caller of these functions across the codebase.
Update each caller to work with the new pattern.

Do NOT change test files yet — we'll handle those separately tomorrow.
```

### Exercise: Verification Pass

Run all three checks:

```bash
# Type checking
npx tsc --noEmit

# Linting
npm run lint

# Build
npm run build
```

Fix issues in batches — don't fix them one by one:

```
Here are all remaining TypeScript errors after the refactor:
[paste all errors]

Fix all of them. Group fixes by file.
```

### Success Criteria for Day 3
- [ ] All files in the inventory are refactored
- [ ] All callers are updated
- [ ] Type checking passes
- [ ] Lint passes
- [ ] Build succeeds

---

## Day 4 (Thursday): Update Tests and Verify

### Morning: Update Existing Tests

```
The refactor changed function signatures and return types.
Update all test files to work with the new pattern.

Rules:
- Keep existing test descriptions and structure
- Update assertions to match new return types
- Add new test cases for any new error paths introduced by the refactor
- Do not change test coverage — every behavior tested before should
  still be tested after

Show changes grouped by test file.
```

Run the tests:

```bash
npm test 2>&1 | head -100
```

For failures, feed them back:

```
These tests are failing after the refactor:
[paste test failures]

For each failure, determine:
1. Is this a test that needs updating (testing the old pattern)?
2. Is this a genuine bug introduced by the refactor?

Fix the test-update cases. For genuine bugs, show me the production
code fix, not a test change.
```

### Afternoon: Manual Smoke Testing

Pick the 3 most critical user flows in your application. Test each one end-to-end:

1. Authentication flow (login, signup, password reset)
2. Core business operation (the main thing your app does)
3. Payment/billing flow (if applicable)

Document what you tested:

| Flow | Steps Tested | Result | Notes |
|------|-------------|--------|-------|
| Auth: login | Email + password login, token refresh | Pass | |
| Auth: signup | New user registration, welcome email sent | Pass | |
| Core: create order | Add items, checkout, confirmation | Pass | |

### Success Criteria for Day 4
- [ ] All tests pass
- [ ] Manual smoke tests pass for top 3 critical flows
- [ ] Any bugs introduced by the refactor are fixed
- [ ] Zero regressions from pre-refactor behavior

---

## Day 5 (Friday): Harness Prompts and Week 3 Metrics

### Morning: Write Reusable Harness Prompts

A "harness prompt" is a prompt template you can reuse for similar tasks in the future. Based on this week's refactor, write 2-3 harness prompts.

### Exercise: Create Your Harness Prompt Library

Create a file in your project (e.g., `prompts/refactor.md`) with templates:

**Template 1: Pattern Inventory**
```
Find every instance of [PATTERN_DESCRIPTION] in this codebase.
For each instance, output:
1. File path and line numbers
2. The exact current code
3. Context: what this instance does
4. Complications: edge cases or special behavior

Output as a numbered list. Be exhaustive.
```

**Template 2: Pattern Application**
```
Apply this transformation to [FILE_LIST]:

BEFORE pattern:
[BEFORE_CODE]

AFTER pattern:
[AFTER_CODE]

Transformation rules:
[RULES]

Exceptions (do NOT change):
[EXCEPTIONS]

For each file, show before/after. Flag non-standard transformations.
```

**Template 3: Post-Refactor Verification**
```
After the refactor:
1. Find any remaining instances of the OLD pattern (should be zero)
2. Find all callers of changed functions and verify they use the new pattern
3. List any type errors, lint errors, or test failures

Output as three sections with file:line references.
```

### Afternoon: Week 3 Metrics

**Refactor Metrics:**
1. Total files changed: ___
2. Total instances transformed: ___
3. Time to complete entire refactor: ___ hours
4. Estimated time without AI: ___ days
5. Non-standard transformations that needed manual attention: ___
6. Bugs introduced by the refactor: ___
7. Bugs caught by type checking: ___
8. Bugs caught by tests: ___
9. Bugs caught by manual testing: ___

**Workflow Metrics (compare to Week 2):**
10. Average time per AI-assisted task: ___ min (Week 2: ___ min)
11. Prompt quality: how often does the first prompt produce usable output? ___% (Week 2: ___%)
12. Tasks where you reverted to manual coding: ___ (should be near zero)

**Qualitative:**
13. What was the hardest part of the refactor? ___
14. Where did AI save the most time? ___
15. Where did AI cause the most problems? ___
16. Would you do this refactor differently next time? How? ___

### Success Criteria for Day 5
- [ ] Harness prompts are written and saved for reuse
- [ ] Week 3 metrics are recorded
- [ ] The refactor is complete, tested, and mergeable

---

## Week 3 Success Criteria (Overall)

You are ready for Week 4 if:

1. **You completed a real codebase-wide refactor using AI.** Not a toy example — a real refactor on a real codebase. 10+ files changed, consistent pattern applied, tests updated, manual testing done.

2. **You can estimate refactor scope accurately.** Given a pattern to change and a codebase, you can inventory instances, estimate time, and execute the plan. The estimation was within 50% of actual time.

3. **You have harness prompts.** Reusable prompt templates for inventory, application, and verification. These are your force multipliers going forward.

4. **You understand when AI needs supervision vs. when to batch.** First 2-3 files: review each one. Remaining files: batch mode. You know the difference and you know when to switch.

5. **Your verification workflow is solid.** Type check, lint, test, smoke test — in that order, after every batch of changes. You don't skip steps.

---

## Common Pitfalls for Week 3

1. **"I'll refactor the whole codebase at once."** Don't. Scope the refactor to one service or one package. A 500-file refactor is not a week-3 exercise — it's a quarter-long project. Pick 10-30 files.

2. **"The AI missed some instances."** Always run a verification pass after the refactor is "done." Ask AI to find any remaining instances of the old pattern. There are almost always 2-3 stragglers.

3. **"Tests pass but behavior changed."** Type checking and unit tests don't catch everything. The manual smoke test on Day 4 is not optional. It catches integration-level issues that unit tests miss.

4. **"I skipped the pattern specification and just started refactoring."** The spec takes 30 minutes to write and saves hours of inconsistency. Without it, the AI applies the pattern slightly differently in each file. With it, every transformation is consistent.

5. **"The refactor PR is too large to review."** If your refactor touches 50+ files, split it into multiple PRs by directory or module. Each PR should be reviewable in 15-20 minutes. A 200-file PR gets rubber-stamped, not reviewed.
