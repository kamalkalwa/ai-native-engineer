# Workflow: Always Have an Agent Running

**Hashimoto's rule.** *"If I'm coding, I want an agent planning. If they're coding, I want to be reviewing."* The wallclock cost of an agent thinking for 30 minutes is zero — if you're not waiting on it. Background agents are how you reclaim the wallclock.

**Time:** 1 hour to set up the queue. The discipline takes a week.
**Tools:** Claude Code (background mode), Cursor Background Agents, Codex Cloud, slow thoughtful models (Amp deep mode, GPT-5.2-Codex).
**Level:** 2 → 3 (presupposes [CLAUDE.md](claude-md-as-kill-log.md) and [harness](harness-engineering.md) are already in place)

---

## Why "always running" is the senior PE move

Most engineers run one agent, watch it, get bored, and lose 10 minutes per task to context-switching. The wallclock is wasted twice — once while the agent thinks, once while you reset focus.

Hashimoto's pattern flips this. **Always have at least one agent running on something.** The rule is simple:

- If you're actively coding, an agent is planning the *next* task.
- If an agent is actively coding, you're reviewing its diff or designing the next task.
- Slower thoughtful models (Amp deep mode, GPT-5.2-Codex) take 30+ minutes per task — perfect for async, useless for interactive.

Hashimoto on Pragmatic Engineer: *"I kick off tasks before leaving the house — research, edge-case analysis, library comparisons — so work progresses while I drive or am away."*

The wallclock budget you used to waste is now a research backlog being burned through in the background.

---

## What goes in the background queue

Not every task is a fit for a slow async agent. The shape of work that benefits:

| Good for background | Bad for background |
|---|---|
| Research ("compare these 3 libraries for our use case") | Anything where you don't know what "done" looks like |
| Edge-case analysis ("list every error mode of this function") | Tight feedback loops (TDD red-green-refactor) |
| Synthesis ("read these 5 PR descriptions, summarize the design intent") | Anything that requires human judgment between steps |
| Test generation against a finished feature | Architectural decisions |
| Documentation drafts | Greenfield API design |
| Refactor exploration ("if I renamed this, where breaks?") | Anything with unbounded scope |
| Spec drafting from a rough description | Code touching auth, billing, payments |
| Dependency audit ("which of our deps haven't been updated in 12 months?") | Migrations |

**The rule:** background tasks should have a *clear definition of done*, no ambiguity, no inter-step decisions. The agent runs, produces output, you read it later.

---

## The minimum viable setup

### Step 1: pick one slow task you'd normally do yourself

Something you keep meaning to get to but don't have a 30-minute block for. Examples:

- "Audit our `package.json` for unused dependencies, list each with evidence."
- "Read the last 20 PR descriptions and summarize what design patterns emerged this quarter."
- "List every place we use `JSON.parse` on external input. For each, suggest whether it should switch to `safeParse`."

### Step 2: write a clear definition of done

The agent needs to know when to stop. Vague: *"clean up the deps."* Clear: *"produce a markdown table with columns `package`, `last-used-in`, `last-updated`, `recommendation: keep | drop`. End with a one-line summary."*

### Step 3: kick it off in a separate worktree

```bash
git worktree add ../proj-bg-deps-audit chore/deps-audit
cd ../proj-bg-deps-audit
claude --task "audit deps, output to deps-audit.md"  # or paste the brief
```

Or use Cursor's Background Agent — it spins up a cloud VM, runs in isolation, opens a PR with the result.

### Step 4: walk away

Drive. Eat. Take a meeting. Sleep. The model's wallclock is spending itself; yours isn't.

### Step 5: review the output later, in a focused block

When you come back, read the output as a *report*, not a *negotiation*. If it's wrong, that's a [harness](harness-engineering.md) signal — what would have prevented it? If it's good, merge or extract the artifact.

---

## The two speeds: fast vs thoughtful models

Hashimoto's pattern explicitly uses two model speeds:

- **Fast iterative models** (Claude Opus 4.7, Sonnet 4.6, Cursor's default) — for the inner loop. Sub-minute responses. You're at the keyboard.
- **Slow thoughtful models** (Amp deep mode, GPT-5.2-Codex, Claude with extended thinking) — for the background. 5–30+ minute responses. Higher quality on hard problems.

The deep models cost more per call but the wallclock is free. Use them on the tasks where quality matters more than latency: research, refactoring strategy, post-mortem analysis, eval-set design.

---

## Three concrete background patterns

### Pattern 1: the morning queue

Before you start coding, kick off 2–3 async tasks. By midday, you have output to review.

```bash
# Morning, 9am — before standup
cd ../proj-bg-research
claude --task "Research: should we migrate from Drizzle to Kysely? Compare on: TypeScript inference quality, SQL escape hatches, migration story, performance, ecosystem. End with a recommendation."

cd ../proj-bg-edges
claude --task "Read app/billing/charge.ts. List every error mode. For each, what does the user see? What does our log show? Which ones are silently swallowed?"
```

By 1pm, both reports are ready. You've shipped your morning's interactive work AND have research for the afternoon's decision.

### Pattern 2: the parallel-with-self pattern

While you actively edit Pane 1, Pane 2 is running an async task related to (but not blocking) your current work.

```
You're refactoring src/auth/oauth.ts (Pane 1, interactive)
Background agent (Pane 2): "Generate test cases for src/auth/oauth.ts based on the spec."
```

When you finish the refactor, the test cases are ready. You review and adapt them. Total wallclock: ~the same as if you'd done the refactor alone.

### Pattern 3: the sleep loop

Long-running tasks that genuinely take an hour run overnight or while you sleep.

```bash
# Friday, 10pm
claude --task "Read every file in src/. Identify dead code (unreferenced exports). Output to deadcode.md. Don't delete anything; just report."
```

Saturday morning, you have a dead-code audit. Friday's wallclock didn't cost you anything.

---

## How this stacks with the other workflows

Always-running is downstream of [CLAUDE.md](claude-md-as-kill-log.md) and [harness](harness-engineering.md). Without those, async agents drift — they have no shared conventions and no validation. Three failed background runs are worse than one failed interactive run because you can't course-correct in real time.

Always-running is upstream of [parallel-agents](parallel-agents.md). Once you're comfortable with one async agent, scaling to 3–5 is mostly an orchestration problem (worktrees, tmux, naming).

The progression:

1. CLAUDE.md (Day 1)
2. Harness (Week 1)
3. One always-running agent (Week 2)
4. Parallel agents (Week 3+)

---

## Common pitfalls

**No definition of done.** A background agent without a clear stopping condition runs until the context window fills, then truncates output unpredictably. **Rule:** every background task ends with *"output to file X with structure Y."*

**Returning to the agent mid-run.** If you peek at a 30-minute background task at minute 10, you've now context-switched into reviewing partial output. The wallclock saving is gone. Walk away until done.

**Using fast models for slow tasks.** The whole point of the background queue is to use the slow, thoughtful models that you can't afford to use interactively. If you're running Sonnet on background tasks, you're getting interactive-quality output and missing the upgrade.

**Treating background output as ground truth.** Background agents miss things. Their output is a *first draft* and a *research artifact*, not a finished decision. Review the same way you'd review a junior engineer's research summary — useful, not final.

**No review block in your day.** If you kick off three background tasks and never schedule the review block, the output piles up unread. Treat the review block as a calendar item.

---

## What to ship this week

**Day 1 (30 min):** identify one task you've been putting off because it needs a 30-minute focused block. Write a clear definition of done. Kick it off in a worktree. Walk away.

**Day 2:** review the output. If it's good, ship the artifact. If it's bad, ask: *would a CLAUDE.md rule have prevented this? Would a harness check have caught it?* Update accordingly.

**Day 3 onward:** every morning, ask *"what's a 30-minute task I can fire-and-forget?"* Kick it off. By end of day 7, you'll have shipped 5+ research artifacts you wouldn't have produced otherwise.

**Week 2:** start running 2 background agents in parallel. See [parallel-agents.md](parallel-agents.md).

---

## Practitioner anchors

- **Mitchell Hashimoto** — coined the *always-running* rule. *"If I'm coding, an agent is planning. If they're coding, I'm reviewing."* Uses Amp deep mode (GPT-5.2-Codex) for 30+ minute thoughtful runs. ([Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/mitchell-hashimoto), [paddo.dev/always-have-an-agent-running](https://paddo.dev/blog/always-have-an-agent-running/))
- **Cursor team** — Background Agents in cloud VMs, opens PRs with screenshots. The hosted version of this pattern. ([Cursor docs](https://docs.cursor.com/en/background-agent))
- **Codex team** — parallel orchestration patterns for long-running Codex agents. ([codex.danielvaughan.com](https://codex.danielvaughan.com/2026/04/18/running-multiple-codex-agents-parallel-orchestration/))

---

## Sources

1. [Always Have an Agent Running — paddo.dev](https://paddo.dev/blog/always-have-an-agent-running/)
2. [Mitchell Hashimoto on Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/mitchell-hashimoto)
3. [Cursor — Background Agents docs](https://docs.cursor.com/en/background-agent)
4. [Cursor Background Agents complete guide — ameany.io](https://ameany.io/cursor-background-agents/)
5. [Mitchell Hashimoto's AI Workflow — serenitiesai.com](https://serenitiesai.com/articles/mitchell-hashimoto-ai-workflow)
6. [Running Multiple Codex Agent Instances — codex.danielvaughan.com](https://codex.danielvaughan.com/2026/04/18/running-multiple-codex-agents-parallel-orchestration/)
