# Workflow: Harness Engineering

**The single highest-compound senior PE move in the agent era.** When an agent makes a mistake, don't correct the output. Engineer the rule the agent self-checks against. Mid PEs correct outputs. Senior PEs build the harness.

**Time:** 30 minutes for the first harness rule. Compounds for the life of the project.
**Tools:** Any agent runner (Claude Code, Cursor, Codex). Pre-commit hooks, validators, lint rules, test scripts.
**Level:** 2 → 3 (this is the structural shift senior PEs are making in 2026)

---

## What "harness" means

Mitchell Hashimoto coined the term *harness engineering* on the [Pragmatic Engineer podcast (April 2026)](https://newsletter.pragmaticengineer.com/p/mitchell-hashimoto). His definition:

> **"Anytime you find an agent makes a mistake, you take the time to engineer a solution such that the agent never makes that mistake again."**

The harness is what surrounds the agent: the AGENTS.md files, the linters, the architectural constraints, the verification scripts, the test infrastructure. The agent is the engine. The harness is the thing that turns engine output into ship-able code.

Hashimoto on Ghostty: each line of his AGENTS.md represents a past agent failure that's now prevented.

---

## Why it works (the OpenAI proof)

OpenAI's Ryan Lopopolo published [*Harness engineering: leveraging Codex in an agent-first world*](https://openai.com/index/harness-engineering/) in 2026. The experiment:

- **5 months**
- **3 engineers**
- **~1 million lines of code** (application logic, infrastructure, tooling, docs)
- **~1,500 PRs merged**
- **3.5 PRs / engineer / day** average throughput
- **Zero manually-written code**

The team's finding, repeated in every secondary writeup: **improving the harness mattered more than improving the model.** The engineers spent their time building verification infrastructure, not writing code. The agents queried traces to verify their own work. Documentation told the agent what to do. Telemetry showed whether it worked. Evals decided whether output shipped.

The senior PE archetype in 2026 isn't writing 1,000 lines of code per day. It's building the harness that lets 3 agents safely write 200,000.

---

## The harness pattern (5 layers)

A complete harness has five layers. Each one prevents a specific failure class. Build them in order; each compounds against the next.

### Layer 1: AGENTS.md / CLAUDE.md as the kill log

Every recurring agent mistake becomes a line in the project's instructions file. See [claude-md-as-kill-log.md](claude-md-as-kill-log.md) for the full pattern. The single highest-leverage harness component, and the cheapest to build.

### Layer 2: Static checks the agent invokes

Lint rules, type checks, schema validators. Fast (sub-second), deterministic, machine-readable. The agent runs them after every change.

```bash
# .claude/hooks/post-edit.sh — runs after every Claude Code edit
#!/bin/bash
set -e

bun typecheck     # TypeScript
bun lint          # ESLint with project rules
bun test:unit     # Fast unit tests
bun check:schema  # Custom schema validators
```

In CLAUDE.md, tell the agent:

```markdown
## After every code change
1. Run `bun check` — it runs typecheck, lint, unit tests, schema validation.
2. If any check fails, fix the issue before proposing the next change.
3. If a check is wrong (false positive), update the check, not the code.
```

### Layer 3: Custom validators for project-specific patterns

The patterns your team learned the hard way that no off-the-shelf linter knows:

- "Never call `db.transaction()` inside a route handler — use `withTransaction(handler)` from `lib/db/middleware.ts`"
- "All API responses must include a `request_id` field"
- "Don't use `JSON.parse` on external input — use `safeParse` from `lib/validation`"

Write these as ESLint custom rules, ts-morph scripts, or simple grep-based pre-commit checks. Each one starts as a CLAUDE.md note and graduates to executable code once the agent has tripped on it twice.

### Layer 4: The verifier loop

After the agent writes code, the agent verifies the code against the spec. Not a human review — an agent loop with a separate context that:

1. Reads the spec
2. Reads the diff
3. Answers: *"Does this diff implement the spec? List anything missing, wrong, or unverified."*

This is what OpenAI's Codex team meant by "agents queried traces to verify their own work." A second agent (or the same agent in a second pass) audits the first agent's output.

```markdown
## Verifier prompt template
You are reviewing a code diff against a spec.

SPEC: {paste spec.md here}
DIFF: {paste git diff here}

For each requirement in the spec, answer:
- Implemented correctly: yes/no/partial
- Test coverage: yes/no/partial
- Concerns: [list any]

End with a single line: SHIP / FIX / REJECT.
```

### Layer 5: Eval set as regression test

For features that depend on probabilistic output (LLM calls, agents, classifiers), the eval set is the test suite. See [Hamel Husain's evals work](https://hamel.dev/blog/posts/evals-faq/) for the canonical pattern.

Three rules:

1. Every prod incident becomes a new eval case the next day.
2. Evals run in CI and block merges.
3. Eval cases are versioned in git. They're the team's institutional memory of *"things that broke once."*

---

## The 50-line starter harness (TypeScript / Bun)

A minimum-viable harness for an LLM-calling function. Schema + retry + validator + fail-loud.

```ts
// lib/harness.ts
import { z } from "zod"
import Anthropic from "@anthropic-ai/sdk"

const client = new Anthropic()
const MAX_RETRIES = 3

export async function harness<T>({
  schema,
  prompt,
  semantic,
}: {
  schema: z.ZodSchema<T>
  prompt: string
  semantic?: (parsed: T) => string | null  // null = ok, string = error message
}): Promise<T> {
  let lastError: string | null = null

  for (let attempt = 0; attempt < MAX_RETRIES; attempt++) {
    const messages = [{ role: "user" as const, content: prompt }]
    if (lastError) {
      messages.push({
        role: "user" as const,
        content: `Previous attempt failed: ${lastError}. Try again, fixing the issue.`,
      })
    }

    const response = await client.messages.create({
      model: "claude-opus-4-7",
      max_tokens: 4096,
      messages,
    })

    const text = response.content[0].type === "text" ? response.content[0].text : ""

    try {
      const parsed = schema.parse(JSON.parse(extractJson(text)))
      const semanticError = semantic?.(parsed) ?? null
      if (semanticError) {
        lastError = semanticError
        continue
      }
      return parsed
    } catch (err) {
      lastError = err instanceof Error ? err.message : String(err)
    }
  }

  // Fail loud — log everything, surface to a human review queue
  throw new HarnessFailure({ prompt, lastError, attempts: MAX_RETRIES })
}

function extractJson(text: string): string {
  const match = text.match(/```json\n([\s\S]*?)\n```/) ?? text.match(/\{[\s\S]*\}/)
  return match ? match[0].replace(/```json\n|\n```/g, "") : text
}

class HarnessFailure extends Error {
  constructor(public details: { prompt: string; lastError: string | null; attempts: number }) {
    super(`Harness failed after ${details.attempts} attempts: ${details.lastError}`)
  }
}
```

Usage:

```ts
const InvoiceSchema = z.object({
  invoice_number: z.string().regex(/^INV-\d{6}$/),
  amount_cents: z.number().int().positive(),
  due_date: z.string().date(),
})

const result = await harness({
  schema: InvoiceSchema,
  prompt: `Extract the invoice fields from: ${rawText}`,
  semantic: (parsed) => {
    if (parsed.amount_cents > 1_000_000_00) return "Amount looks suspiciously large"
    return null
  },
})
```

This is the runtime defense layer — Layer 4 of the harness. Combined with Layers 1–3 (CLAUDE.md, static checks, custom validators), it's a closed loop.

---

## The improvement loop

The harness compounds. Every prod failure → new harness rule → harness gets stronger.

```
Day 1: 50-line harness. 3% of agent calls fail validation.
Week 4: 200-line harness with 12 custom validators. 0.7% fail.
Month 3: 800-line harness, eval set with 87 cases, CI gate. 0.1% fail.
Month 6: The harness is your moat. New hires inherit it. Agents inherit it.
```

After 6 months, the team's harness is a company asset. It encodes every failure mode, every project convention, every architectural decision. New agents (and new humans) skip months of trial and error by inheriting it.

This is what OpenAI meant by "improving the infrastructure around the agent mattered more than improving the model." The model is rented from a vendor. The harness is built by your team. **The harness is the moat.**

---

## Common pitfalls

**The harness becomes the bottleneck, not the multiplier.** If every change requires running a 5-minute harness, agents stall. **Rule:** harness checks must be fast (<10s for the inner loop). Slow checks (full e2e suites, fuzz tests) belong in CI, not the inner loop.

**Treating retry as a fix.** Re-prompting the same model with the same context produces similar output. The retry must include the failure as context. Without that, retries are wishful thinking.

**Validating without semantic checks.** Schema validation catches "this is valid JSON with the right fields." It misses "this email address looks like 'noreply@x' with no real domain." Add a semantic check function alongside every schema.

**Putting harness checks in pre-commit only.** Pre-commit runs once per commit. Agents make 50 changes per session. The harness has to run *between* the agent's changes, not at the end. Use Claude Code hooks or your IDE's file-save hook.

**Building the harness all at once.** A 1,000-line CLAUDE.md written from scratch is read by no agent. Start with 20 lines. Add one line per real failure. The harness earns its weight by being battle-tested.

---

## What to ship this week

**Day 1 (30 min):** Audit your last 10 agent corrections. Pick the 3 you made repeatedly. Write them as CLAUDE.md rules. Commit.

**Day 2 (1 hour):** Pick one CLAUDE.md rule that's expressible as a static check. Convert it to a lint rule, validator, or test. Wire it into your agent's post-edit hook.

**Day 3 (2 hours):** Write the 50-line harness above for one LLM-calling function in your product. Replace the existing try/except. Watch the failure rate.

**Day 4 onward:** Every prod failure → new line in CLAUDE.md or new harness rule. The compound has started.

---

## Practitioner anchors

- **Mitchell Hashimoto** — coined "harness engineering." Every line of Ghostty's AGENTS.md is a past failure. ([Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/mitchell-hashimoto))
- **Ryan Lopopolo (OpenAI)** — *Harness engineering: leveraging Codex in an agent-first world.* 3 engineers, 5 months, 1M LOC, 1,500 PRs, 0% hand-written. ([OpenAI](https://openai.com/index/harness-engineering/))
- **Hamel Husain** — evals as the post-hoc layer of the harness. [evals-skills plugin (March 2026)](https://hamel.dev/blog/posts/evals-skills/).
- **Armin Ronacher** — checkpointed agent loops. Each iteration is recoverable; iteration 7 dies, replays 1–6, continues from 7. ([lucumr.pocoo.org](https://lucumr.pocoo.org/2026/4/4/absurd-in-production/))

---

## Sources

1. [Mitchell Hashimoto's new way of writing code — Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/mitchell-hashimoto)
2. [Harness engineering: leveraging Codex in an agent-first world — OpenAI](https://openai.com/index/harness-engineering/)
3. [Extreme Harness Engineering — Latent Space](https://www.latent.space/p/harness-eng)
4. [Hamel Husain — Evals Skills for Coding Agents](https://hamel.dev/blog/posts/evals-skills/)
5. [Armin Ronacher — Absurd in production](https://lucumr.pocoo.org/2026/4/4/absurd-in-production/)
6. [Instructor library — structured outputs with retry](https://python.useinstructor.com/)
7. [Harness Engineering: The Missing Layer Behind AI Agents](https://www.louisbouchard.ai/harness-engineering/)
