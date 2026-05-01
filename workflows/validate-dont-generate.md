# Workflow: Validate, Don't Generate

**The Sean Goedecke discipline.** A staff engineer on GitHub's Copilot team built his entire AI workflow around a single inversion: *don't let the LLM make the decision; let the LLM verify the decision you already made.*

This is the discipline that survives the perception gap. METR's randomized trial showed engineers feel 20% faster while being 19% slower — because they outsourced the *decision* to the agent and only reviewed the *output*. Validate-don't-generate keeps the decision with you. The agent becomes a verifier, not an author.

**Time:** A behavior change, not a setup. Ship it on the next prompt you write.
**Tools:** Any AI tool. The shift is mental, not technical.
**Level:** 1 → 2 (this is the cheapest taste-preserving move; many senior PEs operate entirely this way)

---

## What "validate, don't generate" actually means

Sean Goedecke, staff engineer on GitHub's Copilot team, summarized his philosophy across multiple interviews ([Rootly](https://rootly.com/humans-of-reliability/sean-goedecke), his own blog at [seangoedecke.com](https://www.seangoedecke.com/)):

> *"The key is to know exactly what you're expecting before asking an LLM to write a snippet for you. I don't want the computer to make decisions for me — I just want AI to type out what I've already devised."*
>
> *"I use AI chats a lot, but never to write code from scratch. Rather, my objective is to validate my approach after I've done writing my solution."*
>
> *"GitHub Copilot is great for generating small snippets, but I never rely on these tools to write big blocks of code."*

The pattern, distilled:

1. **You** decide the approach.
2. **You** write the first draft (or at least the structure).
3. **AI** validates: *"did I miss anything? Is this consistent? Are there edge cases I haven't covered?"*

This is the inversion of how most engineers use AI in 2026. Most prompt: *"implement X."* The agent decides the approach, the structure, the libraries — all the load-bearing decisions. The engineer reviews the output, often in less depth than the agent put into producing it.

Goedecke flips it: *the engineer makes the decisions; the agent stress-tests them.*

---

## Why this is the senior PE move

Senior PE craft is decision-making, not typing. The decisions that matter — architecture, error modes, naming, what's in scope, what's not, what to ship today vs next quarter — are exactly the things that LLMs handle worst and that compound the most over time.

Three reasons validate-don't-generate compounds:

1. **Your taste sharpens.** Every decision you make is reps. Every decision you outsource is reps you don't get. Engineers who outsource decisions to agents stop building taste.
2. **You catch the perception gap.** METR's RCT showed engineers were *slower* with AI but *felt* faster. The gap exists because review is shallower than authorship — you nod along to the agent's output without truly judging it. Authorship-then-validation closes the gap.
3. **The output respects your codebase's decisions.** When you write the structure, the agent fills in details consistent with your structure. When the agent writes the structure, you get whatever its training data thinks is "good" — generic, not specific.

---

## The four moves of the pattern

### Move 1: design before prompting

Before any AI call, write down — even on paper — the answers to:

- What is the function/feature actually doing?
- What are its inputs? Outputs? Side effects?
- What's the failure mode? (What does it look like when this breaks?)
- What pattern in our codebase does this resemble?
- What's *not* in scope?

This is the spec. See [spec-driven.md](spec-driven.md) for the full template. The point is: **you've made the decisions before the agent sees the prompt.**

### Move 2: write the structure yourself

Drop the file. Write the function signature. Sketch the control flow with placeholder bodies. *Then* prompt the agent.

```ts
// You write this scaffolding:
async function chargeWithRetry(
  customerId: string,
  amountCents: number,
  options: { maxRetries?: number } = {}
): Promise<Result<Charge, PaymentError>> {
  // Validate input

  // Idempotency check (look up existing charge)

  // Attempt charge with exponential backoff

  // On final failure, send to dead-letter queue

  // Return Result
}
```

Now prompt: *"Fill in the bodies. Keep the structure. Use the Result type from `lib/result.ts`. Backoff: 1s, 4s, 16s. Max 3 retries on 5xx; never retry 4xx."*

The agent has been pinned. Its job is to fill in implementation details consistent with your structure, not to invent the structure.

### Move 3: ask the agent to find what you missed

After you've written the solution (or the agent has filled in your structure), ask:

> *"Read the function I just wrote. What's missing? List edge cases I haven't handled. List naming inconsistencies with the rest of the codebase. List anything that would surprise a reviewer."*

This is the validation pass. The agent is now in critic mode, not author mode. Critic-mode output is much higher quality than author-mode output — the model is good at finding gaps in existing artifacts, less good at deciding what artifact to build.

### Move 4: use AI for syntax, not structure

Goedecke explicitly says: *"LLMs are great for syntax suggestions or to cover more ground while doing research quickly."* Use them for:

- "What's the TypeScript syntax for a discriminated union with a catchall?"
- "What's the idiomatic way to do X in Drizzle?"
- "Are there 3 alternatives to this approach I should consider?"

These are bounded queries. The decision stays with you; the agent supplies vocabulary and options.

---

## A concrete before/after

### Before (generate-mode)

```
Prompt: "Add payment retry logic with exponential backoff to our checkout."

Agent output: 80 lines of code introducing a new RetryPolicy class, a new
RetryableError exception, a new PaymentRetryService, integrated through
dependency injection (which the codebase doesn't use), with defaults that
don't match the team's conventions. Engineer reviews diff in 4 minutes,
notices 3 issues, asks for revisions, accepts the 3rd version. Total time:
35 minutes. Output: code that mostly works but introduced 4 abstractions
the codebase didn't have.
```

### After (validate-mode)

```
Engineer (10 min):
- Decides: extend existing chargeCustomer() with optional retry config
- Writes function signature + 5-line skeleton
- Prompt: "Fill the bodies. Backoff 1s/4s/16s, max 3 retries on 5xx, never
  4xx. Use existing Result type. No new abstractions."

Agent output (30 sec): 25 lines, fits the codebase conventions.

Engineer review (5 min):
- Reads diff, looks for: "did I miss an edge case?"
- Validation prompt: "What error modes does this code not handle?"
- Agent flags: "doesn't log retry attempts; doesn't propagate request_id
  through retries"
- Engineer adds those (2 min).

Total time: 17 minutes. Output: 25 lines that match the codebase, with
edge cases caught.
```

The validate-mode version is 2x faster, produces less code, and the code that ships matches the codebase's existing decisions.

---

## When to break the pattern

Validate-don't-generate is the senior default, not an absolute rule. Three places where pure-generate is fine:

- **Throwaway prototypes.** If you're going to delete the code in an hour, agent-driven is faster. Don't validate; ship, learn, throw away.
- **Boilerplate where you've already made all the decisions structurally.** Test scaffolding, CRUD endpoints in a well-typed framework, doc updates. Generate freely.
- **Research-mode coding.** *"Show me 3 ways to implement this so I can pick."* You're using the agent as a vocabulary engine. Generate, choose, then move to validate-mode for the chosen path.

The signal that you've drifted into generate-mode where you shouldn't: you're reviewing diffs without strong opinions. *"Looks reasonable, ship it"* is the warning sign. If you haven't formed an opinion before reviewing, you've outsourced the decision.

---

## How this stacks with the other workflows

Validate-don't-generate is the **discipline layer** of the AI-native workflow. The others are infrastructure:

- [CLAUDE.md](claude-md-as-kill-log.md) — encodes your decisions for the agent.
- [Harness](harness-engineering.md) — verifies the agent's output mechanically.
- [Spec-first](spec-driven.md) — formalizes the decision-before-prompting move.
- **Validate-don't-generate** — the daily discipline that prevents you from outsourcing the decisions that matter.

Without this discipline, the other infrastructure compounds in the wrong direction. CLAUDE.md teaches a sloppily-prompted agent to be sloppy faster. The harness catches mechanical errors but not strategic ones. Spec-first only works if you actually wrote the spec.

The taste lives in this file.

---

## Common pitfalls

**Reviewing without an opinion.** If you read agent output and have no prior model of what *should* be there, you can't review — you can only react. **Rule:** before any review pass, write down (in your head or on paper) what the right answer should look like.

**Treating agent confidence as correctness.** Models phrase wrong answers with the same fluency as right ones. Confidence is a feature of the output, not a signal of accuracy. The validation prompt explicitly asks *"what's missing"* because gaps are easier for models to spot in critic mode than to avoid in author mode.

**Skipping the design step "just this once."** Once is twice. Three times is a habit. Senior PEs who drift into generate-mode for "small things" find their decision-making muscle atrophies. The discipline is daily.

**Using validate-don't-generate as a brake on speed.** The pattern is *faster*, not slower, on real work — because it produces output that fits the codebase on the first try. If you find yourself slower, you're probably writing the structure too completely. Sketch the structure (function signatures, control flow), don't write the full implementation.

**Conflating this with "AI is bad."** Goedecke uses AI heavily — daily, hours per day. The pattern isn't AI-skeptic; it's *decision-protective.* The engineer who validates uses AI more than the engineer who generates, because validation is fast and accurate.

---

## What to ship this week

**Day 1:** on your next real coding task, write the function signature and 5-line skeleton *before* opening Claude Code. Prompt with "fill the bodies; don't change the structure." Note how the output compares to your usual generate-mode output.

**Day 2:** add the validation prompt to your habit. After writing any non-trivial code (yours or the agent's), ask *"what's missing? What edge cases haven't I handled?"*

**Day 3:** notice the drift. Each time you typed *"implement X"* without writing structure first, ask why. Add the recurring failure modes to [CLAUDE.md](claude-md-as-kill-log.md) so the agent reminds you.

**Week 2:** measure. Are you shipping more, or fewer, lines of code? More? You may still be over-generating. Validate-don't-generate usually produces *less* code, not more.

---

## Practitioner anchors

- **Sean Goedecke** — staff engineer, GitHub Copilot team. Operates entirely in validate-mode. *"My objective is to validate my approach after I've done writing my solution."* ([seangoedecke.com](https://www.seangoedecke.com/), [Rootly](https://rootly.com/humans-of-reliability/sean-goedecke))
- **Addy Osmani** — *"Never merge code you cannot explain."* The corollary: if you didn't decide it, you can't explain it; if you can't explain it, you shouldn't merge it. ([Addy Osmani — My LLM Coding Workflow Going Into 2026](https://addyosmani.com/blog/ai-coding-workflow/))
- **Simon Willison** — *agentic engineering vs vibe coding.* The professional distinction is "amplifying existing expertise," not "replacing decision-making." ([simonwillison.net](https://simonwillison.net/2026/Feb/23/agentic-engineering-patterns/))
- **METR** — RCT showing the perception gap (engineers feel 20% faster while being 19% slower) is the empirical case for this pattern. ([metr.org](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/))

---

## Sources

1. [Sean Goedecke — Rootly Humans of Reliability](https://rootly.com/humans-of-reliability/sean-goedecke)
2. [Sean Goedecke's blog](https://www.seangoedecke.com/)
3. [Sean Goedecke on the METR study](https://www.seangoedecke.com/impact-of-ai-study/)
4. [Addy Osmani — My LLM Coding Workflow Going Into 2026](https://addyosmani.com/blog/ai-coding-workflow/)
5. [Simon Willison — Agentic Engineering Patterns](https://simonwillison.net/2026/Feb/23/agentic-engineering-patterns/)
6. [METR — Measuring Early-2025 AI on Experienced OSS Devs](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/)
7. [METR — Update on the experiment design (Feb 2026)](https://metr.org/blog/2026-02-24-uplift-update/)
