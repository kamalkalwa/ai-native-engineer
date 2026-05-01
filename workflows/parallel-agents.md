# Workflow: Parallel Agents on Git Worktrees

**The DHH move.** Run two or more agents simultaneously, each in its own working directory, on isolated branches, reviewing each other or working on independent tasks. Doubles wallclock throughput when the work parallelizes. Doubles the chaos when it doesn't.

**Time:** 10 minutes to set up worktrees. The discipline takes weeks.
**Tools:** `git worktree`, tmux or iTerm2 split panes, optional: Rover, Emdash, agent-of-empires, Cursor Background Agents.
**Level:** 2 → 3 (don't attempt this without harness + CLAUDE.md already in place)

---

## Why parallel agents are the senior PE move

The wallclock cost of an agent thinking for 30 minutes is zero — *if you're not waiting on it.* Most engineers run one agent, watch it work, get bored, scroll Twitter, come back to a half-finished diff. The wallclock budget is wasted.

Parallel agents convert wallclock from a bottleneck into a budget. While one agent refactors the auth module, another generates tests for the new endpoint, and a third drafts the migration guide. You become the orchestrator, not the typist.

DHH's setup, observed at 37signals (April 2026): **Gemini K25 in OpenCode on top, Claude Opus in Claude Code on bottom, diffs reviewed in Neovim.** Two models on the same task, reviewing each other's output. The dual-model pattern catches hallucinations that a single model misses.

The community has converged on the practical ceiling: **5–7 concurrent agents on a modern laptop** before merge-review overhead and context-switching tax cancel the throughput gains.

---

## The core problem: agents step on each other

Run two agents in the same working directory and the most common failure mode appears immediately: **silent overwrites.** Agent A edits `src/auth.ts` to add OAuth. Agent B edits `src/auth.ts` to add rate limiting. Agent B's write overwrites Agent A's. Both pass tests independently. The merged state is broken.

`git worktree` solves this at the filesystem level. Each worktree is a separate working directory of the same repository, on a separate branch. Agents in different worktrees can't touch each other's files.

```bash
# Add a worktree for each parallel task
git worktree add ../proj-auth-oauth feature/oauth
git worktree add ../proj-auth-ratelimit feature/ratelimit
git worktree add ../proj-test-gen feature/tests

# Now each directory is a clean checkout on its own branch
ls ..
# proj/                       # main worktree on `main`
# proj-auth-oauth/            # worktree on `feature/oauth`
# proj-auth-ratelimit/        # worktree on `feature/ratelimit`
# proj-test-gen/              # worktree on `feature/tests`
```

Each worktree has its own branch, its own working tree, and its own agent session. They share the `.git` directory but don't touch each other's files.

---

## The minimum viable setup (10 minutes)

### Step 1: enable worktrees in your IDE

- **JetBrains IDEs:** native worktree support shipped in 2026.1 (March 2026).
- **VS Code:** worktree support added July 2025. Install the *Git Worktrees* extension if not bundled.
- **Neovim:** built-in via `:Git worktree` (fugitive) or `gitsigns.nvim`.

### Step 2: set up the tmux layout

```bash
# Start a tmux session named "agents"
tmux new -s agents

# Inside tmux:
# - Split horizontally: Ctrl-b "
# - Split vertically:   Ctrl-b %
# - Move between panes: Ctrl-b arrow
# - Resize:             Ctrl-b Alt-arrow
```

A typical 4-pane layout:

```
+--------------------+--------------------+
| Pane 1: Claude     | Pane 2: Cursor     |
| in proj-oauth/     | in proj-ratelimit/ |
+--------------------+--------------------+
| Pane 3: Tests      | Pane 4: Review     |
| in proj-test-gen/  | watch git log      |
+--------------------+--------------------+
```

### Step 3: launch agents in each pane

```bash
# Pane 1
cd ../proj-auth-oauth && claude

# Pane 2
cd ../proj-auth-ratelimit && cursor .

# Pane 3
cd ../proj-test-gen && claude

# Pane 4 (review pane)
cd ../proj && watch -n 5 'git log --all --oneline -10'
```

### Step 4: brief each agent with a scoped task

Each agent gets a focused, narrowed task. Not "build the auth module" — *"add OAuth login via Google, scoped to `src/auth/oauth.ts` and `src/routes/auth.ts`. Don't touch rate limiting or session handling."*

Scope the agent's surface area. The narrower the surface, the lower the merge-conflict risk later.

### Step 5: integrate

When all agents finish, you merge the branches sequentially:

```bash
git checkout main
git merge feature/oauth
git merge feature/ratelimit  # resolve conflicts if any
git merge feature/tests

# Or use a single integration branch:
git checkout -b feature/auth-integration
git merge feature/oauth feature/ratelimit feature/tests
```

The merge is where the parallel pattern earns or loses its keep. Tasks scoped to disjoint files merge cleanly. Tasks that touched overlapping code force you back into linear conflict resolution.

---

## What parallelizes well, and what doesn't

| Parallelizes well | Doesn't parallelize |
|---|---|
| Different feature areas (auth, billing, search) | Same module, different concerns |
| Test generation for done features | Architecture decisions |
| Doc generation alongside implementation | Schema changes that ripple downstream |
| Refactoring isolated files | Anything touching shared types |
| Dependency upgrades on isolated branches | Migration design |
| Running the same task with different models (DHH dual-model review) | Anything where ordering matters |

**The rule:** parallel agents work when the tasks are *file-disjoint*. The moment two agents need to edit the same file or rely on each other's output, you're doing serial work badly. Switch to one agent.

---

## Two specific patterns worth stealing

### Pattern 1: DHH's dual-model review

Two agents work the **same task** with **different models**. Then they (or you) compare the outputs.

```bash
# Pane 1
cd ../proj-feature-a-gemini && opencode  # using Gemini

# Pane 2
cd ../proj-feature-a-opus && claude       # using Opus
```

Both attempt the same feature. The diffs are reviewed side-by-side in Neovim. The dual-model approach catches:
- Hallucinated API calls (one model invents; the other doesn't)
- Approach mistakes (one picks a brittle pattern; the other picks the canonical one)
- Style drift (one adheres to CLAUDE.md; the other doesn't)

You merge the better of the two, or hand-pick from each. This is expensive in tokens but catches more in review than any single model.

### Pattern 2: Hashimoto's "always one planning, always one reviewing"

> *"If I'm coding, I want an agent planning. If they're coding, I want to be reviewing."* — Mitchell Hashimoto

This is parallelism in time, not space. While you actively edit Pane 1, Pane 2's agent is doing something async — researching a library, analyzing edge cases, drafting tests. When you move to Pane 2, Pane 1 takes over the async work.

You're never waiting. The wallclock budget is fully utilized. See [always-running-agents.md](always-running-agents.md) for the full pattern.

---

## Tools that automate the worktree setup

If managing worktrees by hand becomes a tax, three tools (all 2026, all YC-adjacent) automate it:

- **[Rover](https://www.blog.brightcoding.dev/2026/04/30/rover-the-revolutionary-ai-agent-manager-every-developer-needs)** — creates isolated worktrees per task, spins up containers, installs your chosen agent, runs predefined workflows.
- **[Emdash](https://github.com/generalaction/emdash)** (YC W26) — provider-agnostic desktop app. Run multiple coding agents in parallel, each in its own worktree, locally or over SSH.
- **[agent-of-empires](https://github.com/njbrake/agent-of-empires)** — manage multiple Claude Code, OpenCode, Mistral Vibe, Codex CLI, Gemini CLI, Pi.dev, Copilot CLI, Factory Droid agents from TUI or web. Uses tmux + git worktrees under the hood.

**Cursor Background Agents** (built into Cursor 3.0, early 2026) handle the cloud-VM-per-branch case automatically — each agent runs in an isolated VM, opens a PR, and surfaces a screenshot when done. Cursor SDK opened April 29, 2026.

---

## Common pitfalls

**Spawning too many.** Past 5–7 concurrent agents, the merge-review overhead, context-switching tax, and rate-limit hits cancel out the throughput gain. Most senior PEs land on 2–3 as the steady state.

**Letting agents touch overlapping files.** If two agents both need to edit `src/types.ts`, you've architected the parallelism wrong. Refactor: one agent owns the types change, others rebase against it.

**Skipping the review pane.** A 4th tmux pane that watches `git log` (or a tab open to the GitHub branches view) keeps you anchored. Without it, you lose track of which agent finished what, and PRs sit half-reviewed.

**Treating parallel as faster.** Parallel agents don't make individual tasks faster — they let you do *more tasks at once*. If you only have one task to do, run one agent. Parallelism is for batched work.

**No CLAUDE.md.** Without [CLAUDE.md](claude-md-as-kill-log.md), each agent starts from blank assumptions. Three agents starting from blank assumptions produce three inconsistent diffs. CLAUDE.md is the prerequisite, not an optional extra.

**Skipping the harness.** Three agents producing 3x the output also produce 3x the failures, unless [the harness](harness-engineering.md) catches them. Without harness checks, parallelism is a chaos multiplier.

---

## What to ship this week

**Day 1 (30 min):** add three worktrees for three real branches you have in flight. Run `git worktree add` three times. Open three tmux panes.

**Day 2:** pick one feature with file-disjoint sub-tasks. Run two agents on it (e.g., implementation + tests). Measure time-to-merge vs your baseline.

**Day 3:** if Day 2 went well, try DHH's dual-model pattern: same task, two models, side-by-side review. Pick the better diff.

**Week 2:** install Emdash or set up agent-of-empires if the manual tmux dance gets old.

---

## Practitioner anchors

- **DHH** — Gemini K25 + Claude Opus in parallel tmux. Diffs in Neovim. (37signals, April 2026)
- **Mitchell Hashimoto** — *"If I'm coding, an agent is planning. If they're coding, I'm reviewing."* ([Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/mitchell-hashimoto))
- **Cursor team** — Background Agents in Cursor 3.0, each in a cloud VM with PR + screenshot output. ([InfoQ](https://www.infoq.com/news/2026/04/cursor-3-agent-first-interface/))
- **Codex team** — *Running Multiple Codex Agent Instances: Parallel Orchestration Patterns* ([codex.danielvaughan.com](https://codex.danielvaughan.com/2026/04/18/running-multiple-codex-agents-parallel-orchestration/))

---

## Sources

1. [Git Worktrees for AI Coding — MindStudio](https://www.mindstudio.ai/blog/git-worktrees-parallel-ai-coding-agents)
2. [Parallel Agentic Development with Git Worktrees — MindStudio](https://www.mindstudio.ai/blog/parallel-agentic-development-git-worktrees)
3. [How to Use Git Worktrees for Parallel AI Agent Execution — Augment Code](https://www.augmentcode.com/guides/git-worktrees-parallel-ai-agent-execution)
4. [DHH Goes Agent-First on Everything at 37signals — TeamDay.ai](https://www.teamday.ai/ai/dhh-agent-first-coding-pragmatic-engineer)
5. [Multi-agent Claude Code workflow using tmux — gist](https://gist.github.com/andynu/13e362f7a5e69a9f083e7bca9f83f60a)
6. [Claude Code Multi-Agent tmux Setup — Dariusz Parys](https://www.dariuszparys.com/claude-code-multi-agent-tmux-setup/)
7. [Rover, Emdash, agent-of-empires — see "Tools" section above]
8. [Cursor 3 Introduces Agent-First Interface — InfoQ](https://www.infoq.com/news/2026/04/cursor-3-agent-first-interface/)
