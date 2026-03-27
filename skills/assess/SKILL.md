---
name: assess
description: Diagnose the user's AI-native maturity level by scanning the codebase for evidence and asking 5 scenario questions. Produces a per-dimension workflow profile, not just a score. Use when a developer wants to know where they stand or where to start.
allowed-tools: Read, Grep, Glob, Bash(git *)
---

# AI-Native Engineer Assessment

Run a two-layer diagnostic: codebase evidence scan + scenario questions. Do NOT use self-reported yes/no questions — research shows developers *believe* they're 20% faster with AI while actually measuring 19% slower (METR study, 2025).

## Layer 1: Codebase Scan (run silently — no user interaction needed)

Scan the current repository for these signals. Check each one and note present/absent:

**Workflow configuration:**
- Does `CLAUDE.md` exist at the project root? If yes, is it basic (<20 lines) or structured (rules, constraints, MUST/MUST NOT language)?
- Does `.claude/` directory exist? Any skills, commands, or settings inside?
- Does `.cursorrules` or `.cursor/rules/` exist?
- Any `mcp.json` or MCP server configurations?

**Spec discipline:**
- Search for files named `spec.md`, `SPEC.md`, `prd.md`, `design.md`, `RFC-*`, or similar specification documents
- Search for `REFACTOR_SPEC` or pattern specification files
- Check if there's a `docs/` or `specs/` directory with design documents

**Development patterns (from git log):**
- Run `git log --oneline -50` — any AI co-author commits? (look for "Co-Authored-By" with Claude, Copilot, or AI references)
- Average files changed per commit in last 20 commits? (run `git log --oneline --shortstat -20`)
- Commit frequency — multiple commits per day suggests savepoint discipline

**Testing maturity:**
- Check test-to-source file ratio (glob for test files vs source files)
- Are there test files corresponding to recently changed source files?

**Multi-tool signals:**
- Check for `.github/copilot-*`, `.cursorrules`, `CLAUDE.md`, `AGENTS.md`, `.gemini/` — presence of multiple = multi-tool user

After scanning, hold the results. Don't show them yet — they'll be part of the final profile.

## Layer 2: Scenario Questions

Tell the user:

"I've scanned your codebase. Now 5 quick scenarios — pick the option closest to what you'd actually do. No right answers, just honest instincts."

Ask one at a time. Wait for the answer before the next.

### Q1: Spec Discipline
"You need to add a payment retry system with exponential backoff, max 3 retries, dead-letter queue on final failure. What's your first action?"

- A) Open AI and say "add payment retry with exponential backoff"
- B) Write a spec covering retry intervals, max attempts, error types, idempotency, and failure behavior — then feed that to AI
- C) Search the codebase for existing retry patterns first, then write a spec referencing them, then feed to AI
- D) Implement it manually — this is too critical for AI

Scoring: A=Level 0, B=Level 2, C=Level 3, D=Level 1

### Q2: Context Engineering
"You're working in a 200K-line monorepo. You need to change how one service handles authentication. How do you scope the AI's context?"

- A) Open AI at the repo root and describe the task
- B) Point AI to the specific service directory and the 3-4 relevant auth files
- C) Use a subagent to research the auth flow first, write a summary of findings, then feed that focused context to the implementation agent
- D) Don't use AI for auth — too security-critical

Scoring: A=Level 0, B=Level 2, C=Level 3, D=Level 1

### Q3: Review Process
"AI generates a 400-line diff across 6 files for a feature you requested. How do you review it?"

- A) Run the tests — if they pass, merge
- B) Read the diff line by line, make manual corrections where needed
- C) Scan each file for structural correctness, spot-check edge cases, then feed corrections back as follow-up prompts — iterate until clean
- D) Have a second AI model review the first model's output, then review only the disagreements yourself

Scoring: A=Level 0, B=Level 1, C=Level 2, D=Level 3

### Q4: Failure Recovery
"AI has been iterating on a bug fix for 20 minutes. Each fix introduces a new issue. The code is getting worse. What do you do?"

- A) Give it more context and try again
- B) Fix it manually — AI clearly can't handle this
- C) Revert to last working commit, start a fresh AI session, re-describe only the original error trace and the 2-3 relevant files
- D) Ask AI to explain the root cause without writing any code. Based on the explanation, decide whether to delegate the fix or implement yourself

Scoring: A=Level 0, B=Level 1, C=Level 2, D=Level 3

### Q5: Tool Orchestration
"You have three tasks today: a 50-file logging format migration, a visual bug in a React component, and an architecture decision about database sharding. How do you allocate?"

- A) Same AI tool for all three, one at a time
- B) AI for the migration and visual bug, no AI for the architecture decision
- C) Claude Code in a git worktree for the migration, Cursor for the visual bug (needs inline preview), Claude/ChatGPT web for architecture research — run migration and visual fix in parallel
- D) AI for each task but done sequentially with breaks between

Scoring: A=Level 0, B=Level 1, C=Level 2, D=Level 3

## Scoring and Output

Map each answer to a level (0-3). Present as a **workflow profile**, not a single number:

```
Your AI-Native Workflow Profile
═══════════════════════════════

Spec discipline:      Level [X]  ← [one-line insight based on answer]
Context engineering:  Level [X]  ← [one-line insight based on answer]
Review process:       Level [X]  ← [one-line insight based on answer]
Failure recovery:     Level [X]  ← [one-line insight based on answer]
Tool orchestration:   Level [X]  ← [one-line insight based on answer]

Codebase evidence:
  [✓/✗] CLAUDE.md — [detail]
  [✓/✗] Spec files — [detail]
  [✓/✗] AI co-authored commits — [detail]
  [✓/✗] Test coverage signals — [detail]
  [✓/✗] Multi-tool configs — [detail]

Overall: Level [X] ([Name]) with gaps in [weakest areas]

Highest-ROI next step: [specific recommendation based on weakest dimension]
Install: cp levels/level-[N]-[name].md CLAUDE.md
```

**Recommendation logic (based on weakest dimension):**
- Spec discipline is low → "Start writing specs before prompting. See workflows/spec-driven.md"
- Context engineering is low → "Scope your context, don't dump the whole repo. See workflows/context-engineering.md"
- Review process is low → "Review diffs, not just test results. The 30% rewrite rule. See workflows/test-generation.md"
- Failure recovery is low → "Commit often as savepoints. Fresh session > more context. See migration-plan/week-1.md"
- Tool orchestration is low → "Match tool to task, run independent tasks in parallel. See migration-plan/week-4.md"

Then offer: "Want me to run `/audit` on this codebase? It's a good way to see the workflow in action."
