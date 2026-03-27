# Workflow: Context Engineering

**The skill nobody talks about but everyone needs.** AI quality is bounded by context quality. Dumping your entire repo into the prompt is like giving someone the entire codebase on their first day and saying "fix the auth bug." Context engineering is the art of giving AI exactly the right information — no more, no less.

**Tools:** Claude Code, Cursor, any LLM with file access
**Level:** 2 → 3 (this is the transition behavior)

---

## Why Context Matters More Than Prompts

Most "prompt engineering" advice focuses on how you phrase the request. That matters — but the bigger lever is what information you include with the request.

**Practitioner observation:** "Removing irrelevant context improves performance dramatically. Like it was another world." — commonly reported across developer forums

**The context quality spectrum:**

| Context Quality | What It Looks Like | AI Output Quality |
|---|---|---|
| None | "Add retry logic to the payment service" | Generic code that doesn't match your codebase |
| Dumped | Opens Claude Code at monorepo root, hopes it figures it out | Slow, unfocused, misses conventions, hits context limits |
| Scoped | Points to 3-4 relevant files + describes the task | Good output that follows existing patterns |
| Engineered | Research phase → compressed summary → focused implementation prompt | Best output, fewest iterations, most consistent |

---

## The Three Phases of Context Engineering

### Phase 1: Research (Subagent or Separate Session)

Before asking AI to implement anything, have it research the relevant code:

```
Read the following files and summarize the patterns I need to understand
before making changes to the authentication flow:

- src/middleware/auth.ts
- src/services/authService.ts
- src/models/user.ts
- src/routes/auth.ts

For each file, tell me:
1. What it does (2-3 sentences)
2. What patterns it uses (error handling, validation, response format)
3. What other files it depends on
4. Any implicit rules (naming conventions, import patterns, etc.)

Do NOT suggest changes yet. Just map the territory.
```

**Why a separate phase:** Research pollutes implementation context. If you research and implement in the same session, the AI's context fills with exploration output. By separating them, the implementation agent gets a clean, focused context.

**In Claude Code:** Use subagents for research. They have their own context window and return compressed results.

### Phase 2: Compress (Human + AI)

Take the research output and compress it into the information the implementation agent actually needs:

```
Based on the research above, create a context summary for implementing
[the feature]. Include ONLY:

1. The 2-3 patterns the new code must follow (with file:line examples)
2. The types/interfaces the new code must use
3. The error handling convention
4. Any constraints (don't modify X, must be compatible with Y)

Keep it under 50 lines. This summary will be the context for the
implementation prompt.
```

### Phase 3: Implement (Clean Context)

Start a fresh session (or use a new subagent) with only the compressed context:

```
Context:
[paste the compressed summary from Phase 2]

Task:
[describe what needs to be built]

Spec:
[paste or reference the spec]

Implement this following the patterns and constraints in the context above.
```

This gives the AI exactly what it needs — no noise, no exploration artifacts, no context overflow.

---

## Context Management Techniques

### Technique 1: The 50% Rule

AI quality degrades noticeably past ~50% of the context window. Monitor your usage:

- Claude Code: use `/compact` when you notice quality dropping or after completing a major milestone
- Cursor: start new Composer sessions for new tasks instead of continuing long ones
- ChatGPT/Claude web: start a new conversation when responses get repetitive or off-track

**Signs of context degradation:**
- AI repeats something it already said
- Output quality drops suddenly
- AI "forgets" instructions from earlier in the session
- Bizarre, off-topic suggestions appear

When you see these: compact or start fresh. Don't push through.

### Technique 2: MUST/MUST NOT Over Soft Language

```
# Soft (gets ignored at high context utilization)
Try to use TypeScript strict mode.
Prefer the Result type for error handling.
It would be good to add tests.

# Hard (survives context pressure)
MUST use TypeScript strict mode.
MUST return Result<T, AppError> from all service functions.
MUST NOT use try/catch in service layer — use Result type.
MUST include tests for happy path AND error path.
```

The stronger the language, the more likely it survives when the AI is managing a lot of context.

### Technique 3: Path-Scoped Rules

For projects with different conventions in different areas (frontend vs. backend, API vs. workers), use scoped rules:

**In Claude Code (.claude/rules/):**
```
# .claude/rules/api-rules.md (scoped to src/api/**)
All API endpoints MUST validate input with zod schemas.
All responses MUST use the ApiResponse<T> wrapper.
Error responses MUST include error code, message, and request ID.

# .claude/rules/frontend-rules.md (scoped to src/components/**)
All components MUST use the design system tokens, not raw colors.
All data fetching MUST use the useApi hook, not direct fetch.
```

This way, the AI gets the right rules for the right files automatically.

### Technique 4: Commit-as-Savepoint

Every working state should be committed before asking AI to make more changes:

```bash
# After each successful AI-generated change:
git add src/services/payment.ts src/types/payment.ts
git commit -m "wip: add retry logic for payment charges"

# If the next AI iteration goes wrong:
git checkout -- .   # or git stash
# Start fresh from the last good state
```

This is not just good git hygiene — it's context engineering. A fresh session starting from a clean commit is faster than trying to undo bad AI output in a degraded context.

### Technique 5: Knowledge Base Feedback Loops

When AI makes a mistake, don't just fix it — update the rules so it never happens again:

```
"That error handling is wrong — we use Result types, not try/catch in services."

# Then update CLAUDE.md or .claude/rules/:
MUST NOT use try/catch in service layer. Use Result<T, AppError> pattern.
See src/types/result.ts for the type definition.
See src/services/emailService.ts for a correct example.
```

This is "compounding engineering" — every correction makes the AI permanently better for your project. Over time, your rules file becomes a codified knowledge base of your project's conventions.

---

## Context Engineering for Different Task Types

| Task | Context Strategy |
|---|---|
| **Bug fix** | Error trace + the 2-3 files in the stack trace + recent changes (`git log -5`). Nothing else. |
| **New feature** | Spec + relevant existing patterns (2-3 example files) + types/interfaces. Research phase first. |
| **Refactor** | Pattern inventory (from `/refactor` skill) + target pattern spec + list of exceptions. |
| **Code review** | The diff only. Don't load the whole repo — the diff is the context. |
| **Debugging** | Error trace + what you've already tried + the specific files involved. Exclude everything else. |
| **Documentation** | The source file + its test file + any existing docs. The test file shows intended behavior. |

---

## Common Pitfalls

1. **"I'll just let Claude Code read everything."** It can — but it shouldn't. Reading 200K lines to find the 500 that matter wastes context window and produces unfocused output. Scope first.

2. **"I keep re-explaining the same things."** Put them in CLAUDE.md or `.claude/rules/`. Context that repeats across sessions belongs in persistent configuration, not in prompts.

3. **"The AI forgot what I told it 10 minutes ago."** You've hit context degradation. Compact or start fresh. The information isn't lost — it's just crowded out by newer context. A clean session with a focused prompt recovers instantly.

4. **"I don't know which files are relevant."** That's what the research phase is for. Ask AI: "Which files are involved in [this area]?" Use that to scope the implementation context.

5. **"Subagents feel like overkill."** For small tasks, they are. Use subagents when the task involves understanding a system (auth flow, payment pipeline, data model) before changing it. For scoped single-file changes, just point to the file directly.
