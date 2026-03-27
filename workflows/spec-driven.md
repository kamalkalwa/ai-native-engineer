# Workflow: Spec-Driven Development

**The single highest-leverage behavior change.** Research shows spec-first development is the #1 differentiator between engineers who get value from AI and those who don't. Addy Osmani describes it as "doing waterfall in 15 minutes."

**Time:** 15-30 minutes for the spec. Saves 2-4x that in generation quality.
**Tools:** Any AI tool (Claude Code, Cursor, ChatGPT)
**Level:** 1 → 2 (this is the transition behavior)

---

## Why Specs Change Everything

Without a spec, you're asking AI to read your mind. The output will be generic, require heavy editing, and miss your codebase's conventions.

With a spec, AI has a contract to implement. The output fits your system, handles your edge cases, and follows your patterns — because you specified them upfront.

**The 40/20/40 rule (from practitioner research):**
- 40% of your time: constructing the spec and prompt with context
- 20%: waiting for AI generation
- 40%: reviewing and verifying

If you're spending most of your time on review/fixing, your spec was too thin.

---

## The Spec Template

Create a file called `spec.md` (or `SPEC.md`, or `specs/feature-name.md`) in your project:

```markdown
# Feature: [Name]

## Problem
[What's broken or missing. 2-3 sentences. Include who is affected.]

## Solution
[High-level approach. What will exist after this is done that doesn't exist now.]

## Requirements
- [Specific, testable requirement]
- [Another requirement]
- [Edge case requirement]
- [Error handling requirement]
- [Performance requirement if relevant]

## Non-Requirements (Explicitly Out of Scope)
- [Thing you are NOT building]
- [Thing that might seem related but is a separate task]

## Technical Approach
### Files to Change
- `src/services/payment.ts` — add retry logic
- `src/types/payment.ts` — add RetryConfig type
- `src/config/defaults.ts` — add retry defaults

### Patterns to Follow
- Error handling: follow the Result type pattern in `src/types/result.ts`
- Config: follow the pattern in `src/config/email.ts`
- Tests: follow the structure in `src/services/__tests__/email.test.ts`

### Constraints
- MUST be idempotent (same request retried = same outcome)
- MUST NOT retry 4xx errors (client errors are not transient)
- MUST log each retry attempt with attempt number and backoff duration
- MUST respect existing rate limits on the payment API

## Data Flow
[Optional: describe the data flow if it helps clarify]

```
Request → validate → attempt charge →
  if transient error → wait (exponential backoff) → retry (max 3) →
  if all retries fail → send to dead-letter queue → alert on-call
```

## Acceptance Criteria
- [ ] Retry succeeds on transient 5xx errors
- [ ] No retry on 4xx client errors
- [ ] Exponential backoff: 1s, 4s, 16s
- [ ] Dead-letter queue receives failed charges after max retries
- [ ] Each retry is logged with attempt number and duration
- [ ] Existing tests still pass
- [ ] New tests cover: successful retry, max retries exceeded, non-retryable error
```

---

## Step 1: Write the Spec (15-30 min)

This is human work. AI can help draft it, but you own the decisions.

### If starting from scratch:

Write the spec yourself. Focus on:
1. **What** the feature does (requirements)
2. **What it doesn't do** (non-requirements — prevents scope creep)
3. **Which files** change and which patterns to follow
4. **Constraints** that are non-negotiable (MUST/MUST NOT)
5. **Acceptance criteria** you can test

### If spec-writing feels slow:

Use AI to generate a first draft, then edit:

```
I need to build [feature description].

Generate a spec covering:
1. Requirements (specific, testable)
2. Non-requirements (what's explicitly out of scope)
3. Files that need to change (check the existing codebase)
4. Patterns to follow (find similar features in the codebase)
5. Constraints (what MUST and MUST NOT happen)
6. Acceptance criteria

Output as a markdown spec I can review and edit.
Do NOT write any code yet.
```

Review the draft. The AI will miss:
- Non-requirements (it doesn't know your scope decisions)
- Constraints from team knowledge (rate limits, compliance rules, SLAs)
- Patterns from personal preference vs. codebase convention

Add those. The spec should take 15-30 minutes. This is not wasted time — it's the highest-leverage time you'll spend.

---

## Step 2: Iterate on the Spec (5-10 min)

Before any code is generated, stress-test the spec:

```
Read this spec. Before implementing anything, tell me:

1. What edge cases are missing?
2. What could go wrong that the spec doesn't address?
3. Are there any contradictions or ambiguities?
4. What assumptions are you making that I should confirm?

Do NOT write code. Only identify gaps.
```

Review the AI's questions. Update the spec. Repeat once or twice if needed — the goal is to catch spec gaps before any code exists.

**Why this matters:** Errors in the spec cascade into every generated file. Fixing a spec takes 2 minutes. Fixing generated code takes 30 minutes. Fixing production bugs takes hours.

---

## Step 3: Feed to AI for Implementation

Only now do you ask for code:

```
Read the spec in spec.md.

Implement it following the technical approach and constraints exactly.
For each file, show me the changes.

If any part of the spec is ambiguous, ask me before writing code.
```

The output quality difference between "add payment retry" and a detailed spec is the difference between Level 0 and Level 2.

---

## Step 4: Validate Against Spec

After implementation, validate systematically:

```
Check the implementation against each acceptance criterion in the spec:

For each criterion:
1. Show the specific code that satisfies it
2. If a criterion is not met, show what's missing

Output as a checklist with file:line references.
```

---

## When to Write a Spec vs. When to Skip

| Task | Spec? | Why |
|------|-------|-----|
| New feature (any complexity) | Yes | Specs prevent scope creep and improve AI output |
| Bug fix with known root cause | No | Describe the bug + fix directly |
| Refactor (pattern change) | Yes — as a refactor spec | Define before/after pattern, transformation rules, exceptions |
| Config change / one-liner | No | Overhead exceeds benefit |
| Multi-file change (3+ files) | Yes | Spec ensures consistency across files |
| Security-critical code | Yes + extra constraints | MUST/MUST NOT rules prevent AI from introducing vulnerabilities |

**Rule of thumb:** If the task touches more than 2 files or has more than 1 edge case, write a spec.

---

## Common Pitfalls

1. **"Writing a spec takes too long."** A good spec takes 15-30 minutes. The alternative is 2-3 hours of editing bad AI output. The math always favors the spec.

2. **"The AI ignores parts of my spec."** Use MUST/MUST NOT language, not suggestions. "Prefer TypeScript" gets ignored. "MUST use TypeScript strict mode" gets followed. Also: keep specs under 200 lines — longer specs suffer from the "lost in the middle" problem.

3. **"I don't know the technical approach yet."** Use AI for research first: "What are the common approaches to [problem]? Compare tradeoffs." Then write the spec with your chosen approach. Research and implementation are separate phases.

4. **"The spec becomes outdated during implementation."** Update it as you go. The spec is a living document during the feature, not a contract carved in stone. After shipping, the code is the source of truth — archive or delete the spec.

5. **"My team doesn't use specs."** You don't need team buy-in. Write specs for your own AI workflow. When your output quality visibly improves, the team will notice.
