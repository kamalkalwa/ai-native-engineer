# Week 2: Codebase-Aware Workflows

**Goal:** Move from using AI on isolated tasks to using AI that understands your full codebase. By end of week, AI should be reading your repo context for every multi-file task.

**Level transition:** 1 (AI-Assisted) to 2 (AI-Augmented)

---

## Day 1 (Monday): Codebase Context Prompts

### The Shift

Last week, you described tasks to AI. This week, you teach AI about your codebase before asking it to do anything. The difference:

**Week 1 prompt (context-free):**
```
Write a function to create a new order
```

**Week 2 prompt (codebase-aware):**
```
Read src/services/orderService.ts, src/models/order.ts, and
src/routes/orders.ts.

I need to add a createOrder function. Follow the same patterns as
createUser in src/services/userService.ts:
- Same error handling approach
- Same validation pattern using our zod schemas
- Same return type structure
- Same logging conventions

The order should include: userId, items (array of productId + quantity),
shippingAddress, and paymentMethodId.
```

### Exercise: Rewrite 3 Prompts with Context

Take 3 tasks from today's work. For each one, write the prompt with explicit codebase references:
- Point to existing files that demonstrate the pattern to follow
- Reference your project's specific libraries and conventions
- Mention what NOT to do (patterns that exist in the codebase but are deprecated)

Run each prompt in Claude Code (not ChatGPT or Claude web — Claude Code reads your repo).

### Success Criteria for Day 1
- [ ] Every prompt today included at least 2 file references from your codebase
- [ ] AI output matched your codebase patterns without manual adjustment (for at least 1 task)
- [ ] You noticed the quality difference vs. context-free prompts from Week 1

---

## Day 2 (Tuesday): Multi-File Tasks

### The Rule

Today, only use Claude Code for tasks that touch 2+ files. Single-file changes can use Copilot or Cursor's inline completion.

### Exercise: Cross-Cutting Change

Pick a change that affects multiple files. Common examples:
- Add a new field to a model and update all layers (model, service, route, tests)
- Add logging to 5+ service functions
- Update error messages to follow a consistent format

Use this prompt structure:

```
I need to [describe the change] across multiple files.

Affected areas:
- [model/schema layer]: src/models/___
- [service layer]: src/services/___
- [route/controller layer]: src/routes/___
- [test layer]: src/tests/___

Steps:
1. First, read all affected files and show me the current state
2. Propose all changes as a unified plan (not file-by-file)
3. Show me dependencies: which changes must happen before others
4. Implement all changes

Constraints:
- [any backward compatibility requirements]
- [any files to NOT change]
```

### Track: File Count

Record how many files you changed today with AI assistance. By end of Week 2, you should be comfortable with 5-10 file changes in a single AI session.

### Success Criteria for Day 2
- [ ] You completed at least one task that changed 3+ files using Claude Code
- [ ] The AI maintained consistency across all changed files (same patterns, same naming)
- [ ] You described the change as a system change, not a per-file change

---

## Day 3 (Wednesday): AI-Assisted Debugging

### The Debugging Workflow

This is one of the highest-ROI AI workflows. The prompt structure:

```
Here's the error I'm seeing:
[paste full error trace — all of it, not just the message]

This happens when: [exact reproduction steps]
It started after: [what changed — a deploy, a PR merge, a dependency update]
I've already checked:
- [thing you checked and ruled out]
- [another thing you checked and ruled out]

Read the relevant files and explain:
1. Root cause
2. Why the recent change triggered it (if applicable)
3. The fix, with exact diffs
4. How to prevent this class of bug in the future
```

### Exercise: Debug a Real Issue

Find a bug in your tracker — ideally one that's been open for a while, or one you've been avoiding because it's in unfamiliar code.

Use the debugging prompt above. Key points:
- Paste the FULL error trace. AI needs the stack trace to trace the code path.
- Include what you've already tried. This prevents the AI from suggesting the first 3 things you already checked.
- Ask for the "why" not just the fix. Understanding the root cause is the skill you're building.

### Exercise: Reproduce and Fix in One Session

Take the debugging session end-to-end:
1. Describe the bug to Claude Code
2. Let it identify the root cause
3. Let it propose and implement the fix
4. Run the tests
5. If tests fail, feed the failures back to Claude Code
6. Commit and create a PR

**Target time:** Under 30 minutes for a medium-complexity bug.

### Success Criteria for Day 3
- [ ] You debugged at least one issue using the structured debugging prompt
- [ ] The AI identified the root cause correctly (or got you 80% of the way there)
- [ ] You included "already checked" in your prompt and noticed the AI didn't repeat those suggestions

---

## Day 4 (Thursday): AI Test Generation

### The Goal

Today, every PR you create or work on must include AI-generated tests.

### Exercise: Generate Tests for Your Current Work

For whatever you're working on today, generate tests using this prompt:

```
Generate tests for [file or module].

Rules:
1. Test BEHAVIOR, not implementation
2. Each test name completes "it should ___" with something a product
   manager would understand
3. Test happy path, invalid input, edge cases, and error conditions
4. Mock external dependencies only (database, APIs, file system)
5. Assert on complete return values, not just existence

Testing framework: [jest/vitest/pytest]
```

### Exercise: Track Your Rewrite Rate

After the AI generates tests:
1. Count the total assertions
2. Read every assertion
3. Mark which ones test implementation (mock call counts, internal state, snapshot)
4. Rewrite those to test behavior
5. Calculate: `rewrite_rate = assertions_rewritten / total_assertions`

**Target rewrite rate:** 20-35%. If it's higher, your prompt needs more behavior-focused rules. If it's lower, you might be accepting assertions that test implementation.

| Module | Total Assertions | Rewritten | Rewrite Rate |
|--------|-----------------|-----------|-------------|
| Module 1 | __ | __ | __% |
| Module 2 | __ | __ | __% |

### Success Criteria for Day 4
- [ ] You generated tests for at least one module using AI
- [ ] You reviewed every assertion (not just ran the tests)
- [ ] You tracked your rewrite rate and it's in the 20-35% range
- [ ] Your PR includes AI-generated, human-reviewed tests

---

## Day 5 (Friday): Full PR Workflow and Week 2 Metrics

### Morning: End-to-End PR Workflow

Complete one PR using the full AI-assisted workflow:

1. **Describe the task** with codebase context references
2. **Generate the implementation** — review, iterate, finalize
3. **Generate the tests** — review assertions, rewrite 30%, add missing edge cases
4. **Generate the PR description** — including risk assessment and rollback plan
5. **Run the self-review prompt** — check for security, performance, and edge cases

Time yourself. Record each step's duration.

| Step | Time |
|------|------|
| Task description + approach review | __ min |
| Implementation generation + review | __ min |
| Test generation + review | __ min |
| PR description generation | __ min |
| Self-review | __ min |
| **Total** | __ min |

### Afternoon: Week 2 Metrics

**Quantitative:**
1. Tasks completed with codebase-aware prompts: ___
2. Multi-file tasks completed with AI: ___
3. Bugs debugged with AI debugging workflow: ___
4. Test suites generated with AI: ___
5. Average rewrite rate on AI-generated tests: ___%
6. Average prompt iterations before acceptable output: ___
7. Time for full PR workflow (today's exercise): ___ minutes

**Compare with Week 1:**
8. Tasks where AI output matched codebase patterns on first try: ___% (Week 1: likely <30%, Week 2 target: >60%)
9. Times you reverted to manual coding: ___ (should be decreasing)
10. Average time per task (AI-assisted): ___ min (compare to Week 1 baseline)

**Qualitative:**
11. What changed most in how you write prompts? ___
12. Where does AI still waste more time than it saves? ___
13. What's your biggest remaining friction point? ___

### Success Criteria for Day 5
- [ ] Completed one full PR workflow (implementation, tests, description, self-review)
- [ ] Week 2 metrics are recorded and compared against Week 1
- [ ] You identified specific areas for Week 3 improvement

---

## Week 2 Success Criteria (Overall)

You are ready for Week 3 if:

1. **Codebase context is automatic.** Every prompt includes file references, pattern references, and constraint awareness. You don't write context-free prompts anymore.

2. **Multi-file tasks use AI.** You default to Claude Code for any task touching 2+ files. Single-file changes use Copilot/Cursor inline.

3. **You have a debugging workflow.** When you hit a bug, your first instinct is to paste the error trace into Claude Code with context, not to set breakpoints and step through manually. (Both are valid — but AI should be your first attempt.)

4. **AI test generation is part of your PR process.** Every PR includes AI-generated, human-reviewed tests. You know your rewrite rate and it's in the 20-35% range.

5. **You can do a full PR workflow in under 2 hours** (for a medium-complexity feature). Implementation, tests, description, and self-review — all AI-assisted.

---

## Common Pitfalls for Week 2

1. **"Claude Code is slow to read the whole repo."** For large repos, scope it: "Read only the files in src/services/ and src/models/ for this task." Don't make it read your entire monorepo for a scoped change.

2. **"The AI keeps suggesting patterns that don't match our codebase."** You're missing context in your prompt. Always include: "Follow the pattern in [specific file]" and "We use [specific library] for this, not [common alternative]."

3. **"I spent more time writing the prompt than coding would have taken."** This is normal for Days 1-2. By Day 4, your prompt-writing speed should match or exceed your coding speed for the same task. If not, you're over-specifying — give the AI room to figure out obvious things.

4. **"AI-generated tests all pass but don't catch bugs."** You're accepting weak assertions. `toBeDefined()` and `toBeTruthy()` pass on anything. Rewrite to assert on specific values: `toEqual(expectedObject)`.

5. **"I don't know what rewrite rate to target."** If you're rewriting >50% of assertions, your prompt needs better behavior-testing rules. If you're rewriting <15%, you might be missing implementation-testing assertions. 20-35% is the sweet spot.
