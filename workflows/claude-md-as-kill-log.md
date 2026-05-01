# Workflow: CLAUDE.md as a Kill Log

**The single highest-leverage 30 minutes you can spend on your AI workflow.** A CLAUDE.md file at your project root that the agent reads on every session. Karpathy's pattern took agent accuracy from **65% to 94%**. The viral version got 48,900 GitHub stars in a week.

The trick: don't write CLAUDE.md as documentation. Write it as a **kill log** — every line is a past agent failure that's now prevented.

**Time:** 30 minutes for a starter file. Compounds for the life of the project.
**Tools:** Claude Code, Cursor (`.cursorrules`), Codex (`AGENTS.md`), any agent that reads project instructions.
**Level:** 1 → 2 (this is the cheapest big-win in the repo)

---

## Why CLAUDE.md is the highest-ROI file in your project

Every agent session starts from zero. The agent doesn't know your stack, your conventions, your team's gotchas, the libraries you've ruled out, the patterns you stopped using last quarter. Without a CLAUDE.md, you re-explain those things every session, in chat, and most of your corrections die when the session closes.

CLAUDE.md is the *one file* the agent reads on every session. It's loaded into context automatically. Anthropic's [Claude Code best practices](https://code.claude.com/docs/en/best-practices) treat it as the canonical project-instructions surface. Cursor reads `.cursorrules`. Codex reads `AGENTS.md`. The Skills 2.0 release (April 2026) unified custom commands and skills under the same convention.

**Karpathy's CLAUDE.md** (technically [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) — distilled from Karpathy's viral observations on LLM coding pitfalls) hit 48,900 stars by being **short, opinionated, and failure-led.** Not a 500-line style guide. Four core rules and a transformation pattern.

---

## The two patterns that work

### Pattern 1: Karpathy's four rules (failure-mode-led)

Karpathy's CLAUDE.md is built around the failure modes he observed:

1. **Don't assume.** State assumptions explicitly. If uncertain, ask.
2. **Don't hide confusion.** Surface what you're unsure about; don't paper over it with confident-sounding code.
3. **Expose tradeoffs.** Don't pick one option silently — name the alternatives and the reason for the choice.
4. **Surgical changes only.** Don't "improve" adjacent code. No features beyond what was asked.

Plus the transformation rule: **convert imperative tasks into verifiable goals before starting.** Example:

> "Fix the bug" → "Write a test that reproduces it, then make it pass."

This pattern is failure-led: each rule corresponds to a known LLM pathology (silent wrong assumptions, scope creep, hidden uncertainty).

### Pattern 2: Hashimoto's kill log (incident-led)

Mitchell Hashimoto's [Ghostty AGENTS.md](https://newsletter.pragmaticengineer.com/p/mitchell-hashimoto) is built differently. **Every line corresponds to a real past mistake the agent made on this project.**

The pattern, as he describes it on Pragmatic Engineer: *"Anytime you find an agent makes a mistake, you take the time to engineer a solution such that the agent never makes that mistake again."*

Concretely, when an agent suggests Express in a Hono codebase for the third time:
- **Mid PE move:** correct it in chat. Mistake repeats next session.
- **Senior PE move:** add a line to AGENTS.md: *"This codebase uses Hono. Never suggest Express, even for new endpoints."*

Each line of AGENTS.md is now incident-shaped. The file is a forensic log of every recurring failure, with a rule that prevents the next occurrence.

---

## The combined template

The strongest CLAUDE.md combines both: **Karpathy's structural rules at the top, Hashimoto's project-specific kill log below.** Here's a starter you can paste and adapt.

```markdown
# CLAUDE.md

## Voice and behavior

1. **Don't assume.** State assumptions explicitly. If uncertain, ask.
2. **Don't hide confusion.** If you're unsure why something works, say so.
3. **Expose tradeoffs.** When you pick an approach, name the alternatives and why.
4. **Surgical changes.** Touch only what was asked. Don't "improve" adjacent code.
5. **Verifiable goals.** Before "fix the bug," write a test that reproduces it.

## Stack

- Runtime: Bun 1.1+
- Web: Hono (NOT Express)
- DB: Postgres via Drizzle ORM (NOT Prisma)
- Validation: Zod
- Tests: Bun's built-in test runner

## Conventions

- File naming: `kebab-case.ts` for files, `PascalCase` for components, `camelCase` for functions
- Error handling: Result type pattern from `lib/result.ts` (NOT throw/catch in route handlers)
- API responses: every response includes `request_id` field
- Config: env vars validated with Zod at boot in `lib/env.ts`

## Non-negotiables

- MUST use `withTransaction()` from `lib/db/middleware.ts` for any multi-statement DB write.
- MUST NOT call `JSON.parse` on external input — use `safeParse` from `lib/validation`.
- MUST NOT introduce a new dependency without asking. Confirm before `bun add`.
- MUST run `bun check` after every code change. If a check fails, fix the issue before the next change.

## Kill log

Each line below is a past agent failure that's now prevented. Adding a new line is the cost of a recurrent mistake.

- 2026-04-12: Don't suggest Express. This codebase has been on Hono since 2025-Q4.
- 2026-04-15: Don't add `try/catch` around route handlers — use the global error middleware in `app/middleware/errors.ts`.
- 2026-04-18: Don't use `useState` for derived state; use `useMemo`. We had a bug where the derived value drifted.
- 2026-04-22: When generating tests, don't mock our own DB layer. Use the test container in `tests/setup.ts`. Mocking the DB hides bugs in our own ORM usage.
- 2026-04-26: Don't run migrations from the agent session. Migrations go through `bun run migrate` and a human reviews the SQL.
- 2026-04-29: Don't use `JSON.parse(req.body)` — Bun gives you `await req.json()` already validated. Doing it manually broke our error handler.

## Tools the agent should use

After every code change:
1. Run `bun check` — typecheck, lint, unit tests.
2. If a check fails, fix the issue before the next change.
3. If the check itself is wrong, update the check, not the code.

For multi-file changes, use:
- `bun ai:scope` — prints the files relevant to a feature based on `.specs/feature-name.md`
- `bun ai:verify <feature-name>` — runs the verifier loop against the spec

## What NOT to delegate to the agent

- Architecture decisions on new modules — propose, but don't implement until I review.
- Anything in `app/billing/` or `app/auth/` — propose only.
- Database migrations — never run autonomously.
- Production secrets — never read, never log.
```

---

## How to actually build this (the 30-minute starter)

**Step 1 (5 min): copy the template above.** Adapt the stack section to your actual stack. Be specific. *"Bun + Hono + Drizzle"* is useful; *"TypeScript with Node"* is not.

**Step 2 (10 min): mine your last 10 agent corrections.** Open your last few Claude Code or Cursor sessions. Find the moments you said *"no, do it this way instead."* Each one is a kill-log line. Paste them in.

**Step 3 (5 min): add 3 non-negotiables.** Things that, if violated, break production. Phrase them as MUST / MUST NOT, not as preferences. Karpathy-style imperatives — agents respect imperative language more than suggestions.

**Step 4 (5 min): name what NOT to delegate.** Auth, billing, payments, migrations — anywhere the cost of a wrong agent edit is real. The agent should propose, you should implement.

**Step 5 (5 min): commit and run.** `git add CLAUDE.md && git commit -m "add agent instructions"`. Open a fresh Claude Code session. Watch the agent reference your conventions on the first prompt.

Total: 30 minutes. Highest single-action lift in this repo.

---

## The improvement loop

This is the heart of the pattern. **CLAUDE.md is a living file.** Update it the moment you make a correction.

```
Day 1: 30-line CLAUDE.md from the starter template. Agent uses it.
Week 2: 60 lines. Each new line is a real correction you made.
Month 1: 120 lines. Sections start to crystallize (auth, db, tests).
Month 3: 200 lines. Hashimoto's rule kicks in — beyond 200, split into a `.claude/skills/` folder with one SKILL.md per workflow. Skills 2.0 (April 2026) makes this clean.
Month 6: New hires read CLAUDE.md and onboard 2x faster. The file *is* the team's institutional memory.
```

When a CLAUDE.md exceeds ~200 lines, agent context starts paying token tax. That's the signal to graduate sections into Skills folders. See the [Anthropic Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) — each skill is a folder with `SKILL.md` and optional bundled scripts/templates. Loaded only when relevant.

---

## Common pitfalls

**Writing it as documentation, not as a kill log.** A polished, comprehensive CLAUDE.md written from scratch is read by no agent — it's too generic. The kill log gets read because every line carries weight. **Rule:** if a line wasn't earned by a real failure, it doesn't belong in CLAUDE.md.

**Politeness over imperative.** Agents respect "MUST NOT use Express" more than "we usually prefer Hono." The capitalization matters. So does the negation. Karpathy's rules are imperative because imperatives compress better in attention.

**Adjacent improvement creep.** The agent will sometimes "improve" your CLAUDE.md unprompted (rephrase, reorganize, expand). Don't accept those changes by default. The file's value is its lived history; reorganization breaks the pattern.

**Letting it sprawl past 200 lines.** Past ~200 lines, the file becomes a token sink and the agent stops reading carefully. Graduate older sections into `.claude/skills/<name>/SKILL.md` files that load only when triggered.

**Treating it as one-shot.** A CLAUDE.md committed and forgotten compounds zero. The compound only happens if you treat it as a *living* file — every correction is a candidate for a new line.

---

## What to ship this week

**Today (30 min):** copy the template above. Adapt to your stack. Mine your last 10 corrections. Commit `CLAUDE.md`.

**Tomorrow:** every time you correct the agent, ask yourself *"is this a recurring failure? Should this be in CLAUDE.md?"* If yes, add it before the session ends.

**Day 7:** look at your CLAUDE.md. It should have grown by 5–10 lines. The growth rate is the measurement that matters.

**Day 30:** if the file is past 150 lines and growing, start factoring sections into Skills folders.

---

## Practitioner anchors

- **Andrej Karpathy** — original viral observations on LLM coding pitfalls. Forrest Chang's distillation got 48,900 stars. Took accuracy 65% → 94%. ([SOTAAZ writeup](https://www.sotaaz.com/post/karpathy-claude-md-en))
- **Mitchell Hashimoto** — the kill-log pattern on Ghostty. *"Each line is a past failure prevented."* ([Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/mitchell-hashimoto))
- **Anthropic** — official best practices. Skills 2.0 unifies CLAUDE.md, AGENTS.md, custom commands, and skills under one convention. ([Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview))
- **Pere Villega** — *What's New in Claude Code Skills 2.0* (April 1, 2026). When CLAUDE.md outgrows itself, factor into Skills. ([perevillega.com](https://perevillega.com/posts/2026-04-01-claude-code-skills-2-what-changed-what-works-what-to-watch-out-for/))

---

## Sources

1. [Why Karpathy's CLAUDE.md Got 48K Stars — SOTAAZ](https://www.sotaaz.com/post/karpathy-claude-md-en)
2. [Karpathy's CLAUDE.md Skills File — antigravity.codes](https://antigravity.codes/blog/karpathy-claude-code-skills-guide)
3. [Mitchell Hashimoto on Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/mitchell-hashimoto)
4. [Anthropic — Best Practices for Claude Code](https://code.claude.com/docs/en/best-practices)
5. [Anthropic — Equipping Agents for the Real World with Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
6. [Pere Villega — What's New in Claude Code Skills 2.0](https://perevillega.com/posts/2026-04-01-claude-code-skills-2-what-changed-what-works-what-to-watch-out-for/)
