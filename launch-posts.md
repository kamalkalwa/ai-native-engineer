# Launch Posts for AI-Native Engineer

All posts ready to copy-paste. Each section includes the exact text, platform-specific notes, and hashtags/tags where relevant.

---

## 1. Hacker News (Show HN)

### Title

```
Show HN: AI-Native Engineer – Installable Claude Code skills and a scenario-based workflow diagnostic for engineers adopting AI tools
```

### Comment (post this as the first comment)

```
I built this because most "AI for developers" content is either vibes-based advice or tool-specific tutorials that don't transfer.

What it actually is: 5 Claude Code skills you can install in 60 seconds (codebase audit, test generation, multi-file refactor, PR workflow, and a workflow diagnostic). Copy a folder, run a slash command, get output.

The diagnostic part is deliberately scenario-based rather than self-reported. The METR study (2025) found that developers believed they were 20% faster with AI while actually measuring 19% slower. So instead of asking "do you write specs?" it presents situations and asks what you'd do.

There are also three drop-in CLAUDE.md configs (assisted, augmented, native) that change how the AI behaves — Level 1 explains everything before acting, Level 3 decomposes tasks in parallel with minimal confirmation.

Repo: https://github.com/kamalkalwa/ai-native-engineer

Interested in feedback on the skill design and whether the diagnostic dimensions (spec discipline, context engineering, review process, failure recovery, tool orchestration) match what people are actually seeing as differentiators.
```

### Posting Notes
- Best time: Weekday morning US Eastern (8-10am ET), which is evening IST
- Do NOT include "I'm a senior engineer" or credentials in the comment — HN will downvote self-promotion
- The title should be factual and understated. HN prefers "what it does" over "why it matters"
- If it gains traction, be ready to respond to comments within the first 2 hours — HN engagement window is short

---

## 2. Reddit r/ExperiencedDevs

### Title

```
The gap between engineers who use AI tools and engineers who use them well is widening — here's the framework I use to think about it
```

### Body

```
I've been thinking about this for a while and wanted to share the mental model I landed on, since it shifted how I approach my own workflow.

**The core observation:** Most teams have adopted AI coding tools. Copilot, Cursor, Claude Code — adoption is widespread. But the productivity distribution is bimodal. Some engineers are measurably faster on multi-file tasks. Others have the same tools installed and see marginal benefit.

After going through a lot of practitioner reports (HN threads, Reddit posts, Addy Osmani's writing, the Thoughtworks radar), I think the gap comes down to 5 specific skills, not tool choice:

**1. Spec discipline**
The #1 differentiator in every practitioner report I've read. Engineers who write a spec before prompting get usable output on the first try. Engineers who type "add payment retry with exponential backoff" get generic code that takes 3-4 iterations. The rule of thumb I've landed on: 40% writing the spec, 20% waiting, 40% reviewing.

**2. Context engineering**
"Removing irrelevant context improves performance dramatically" — this keeps showing up. The mistake is dumping the whole repo into AI's context window. The skill is scoping: point it at the 3-4 relevant files, or better yet, use a subagent to research first and compress the findings.

**3. Review process**
The METR study (2025) found developers believed they were 20% faster with AI while actually measuring 19% slower. The hypothesis: they were spending review time poorly. Running tests and merging is Level 0. Reading diffs line-by-line is Level 1. The effective pattern seems to be: structural scan, spot-check edge cases, feed corrections back as follow-ups, iterate until clean.

**4. Failure recovery**
When AI spirals on a bug fix — each attempt introduces a new issue — the instinct is to give it more context. The effective pattern is the opposite: revert to last known good, start a fresh session, describe only the original error and the minimal relevant files. Sunk cost applies to AI sessions too.

**5. Tool orchestration**
Not "which tool is best" but matching tool to task. A 50-file migration belongs in Claude Code. A visual React bug belongs in Cursor where you can see inline preview. Architecture decisions belong in a research conversation, not a code-generation session. Independent tasks run in parallel.

**The level model:**
- Level 0: No AI tools
- Level 1: Autocomplete (Copilot tab-completion). ~20-30% faster on typing. Workflow unchanged.
- Level 2: Collaborator. Describe tasks before coding. Multi-file changes. Review and iterate. ~2x on multi-file tasks.
- Level 3: Orchestrator. Decompose features into parallelizable sub-tasks. Run multiple agents. Measure by decisions made, not code written.

The uncomfortable part: the restructuring math. If 7 AI-native engineers produce the same output as 10 traditional engineers, budget-constrained teams will make the obvious move. The question isn't whether this happens — it's which side of that equation each of us is on.

**What I built from this:**
I turned this framework into an open-source tool: installable Claude Code skills (codebase audit, test generation, multi-file refactor, PR workflow) and a scenario-based diagnostic that maps you across those 5 dimensions. It also includes drop-in CLAUDE.md configs that change how AI tools behave at each level.

The diagnostic deliberately avoids self-reported questions for the reason mentioned above (METR study). It presents scenarios and asks what you'd do.

https://github.com/kamalkalwa/ai-native-engineer

Curious if these 5 dimensions match what others are seeing, or if there are gaps. Especially interested in hearing from people who've been through a team restructuring that was explicitly AI-driven.
```

### Posting Notes
- Best time: Tuesday-Thursday, 9-11am ET
- r/ExperiencedDevs requires substantial text posts — no link-only posts
- The value is in the framework, not the repo. The repo is mentioned once at the end.
- Flair: if available, use "General Discussion" or similar
- Do NOT cross-post to r/cscareerquestions — different audience, different tone

---

## 3. Reddit r/programming

### Title

```
I built installable Claude Code skills that encode full engineering workflows as slash commands — codebase audit, test generation, multi-file refactor, PR workflow
```

### Body

```
The idea: instead of prompting AI from scratch every time, encode your best workflows as installable skills. Copy a folder into `.claude/skills/`, and it becomes a slash command.

**What the skills do:**

- `/audit` — Scans for security risks, dead code, performance issues, type safety gaps, error handling gaps, test coverage holes, dependency health. Outputs severity-rated tables with file:line references.

- `/test-gen` — Generates behavior-based tests with guardrails against the common AI test anti-patterns: testing mocks instead of code, weak assertions (toBeDefined/toBeTruthy), missing error paths. Flags assertions that need human review.

- `/refactor` — Guided multi-file refactor: inventory all instances, define target pattern, supervised application on first 3 files, batch the rest, verification pass.

- `/pr` — End-to-end: implementation review, test generation, self-review (security + performance + edge cases), PR description with risk assessment and rollback plan, commit and push.

- `/assess` — Scenario-based workflow diagnostic. Scans your codebase for signals (CLAUDE.md, spec files, AI co-authored commits, test coverage) then asks 5 scenarios. Maps you across 5 dimensions instead of giving a single score.

**Try it in 60 seconds:**

```bash
git clone https://github.com/kamalkalwa/ai-native-engineer
cp -r ai-native-engineer/skills/audit your-project/.claude/skills/audit
cd your-project && claude
# Then type: /audit
```

Also includes 3 drop-in CLAUDE.md configs (assisted, augmented, native) that change how AI tools behave — Level 1 coaches you, Level 3 decomposes tasks and runs with minimal confirmation.

https://github.com/kamalkalwa/ai-native-engineer
```

### Posting Notes
- Best time: Weekday mornings ET
- r/programming prefers technical specifics over thought leadership
- The code block showing "try it in 60 seconds" is the hook — make sure Reddit formatting renders it correctly
- Tags/flair: use whatever is closest to "Tools" or "AI/ML" if available

---

## 4. LinkedIn

### Post

```
A recent study by METR (2025) found something uncomfortable: developers believed they were 20% faster with AI coding tools, while the actual measurement showed they were 19% slower.

The gap isn't whether engineers use AI tools — it's how they use them.

After studying practitioner reports from experienced engineers (HN discussions, Reddit threads, Addy Osmani, Thoughtworks), a pattern emerged. The engineers who are measurably more productive share 5 specific skills:

1. Spec discipline — they write requirements before prompting
2. Context engineering — they scope what the AI sees, not dump the whole repo
3. Review rigor — they iterate on diffs, not just run tests and merge
4. Failure recovery — they know when to revert and start fresh instead of pushing through
5. Tool matching — different tools for different task types, run in parallel

The engineers who installed Copilot and stopped there? Still at autocomplete level. The gap is growing.

I built an open-source tool to help engineers make this transition systematically:

- 5 installable Claude Code skills (codebase audit, test generation, multi-file refactor, PR workflow, workflow diagnostic)
- 3 drop-in config files that change how AI tools behave at different skill levels
- A scenario-based diagnostic that maps you across those 5 dimensions
- A 4-week migration plan with daily exercises

The diagnostic avoids self-reported questions (for the exact reason the METR study revealed). It presents real scenarios and asks what you'd do.

Install a skill, run it on your codebase, see the value in 60 seconds. No signup, no paywall.

Link in comments.

If you manage an engineering team, this might be worth sharing. The skills gap between AI-aware and AI-native is becoming a hiring signal.
```

### First Comment

```
Repo: https://github.com/kamalkalwa/ai-native-engineer

Try the browser-based diagnostic (90 seconds): https://kamalkalwa.github.io/ai-native-engineer/assess/
```

### Posting Notes
- Best time: Tuesday-Thursday, 8-10am in your target audience's timezone (US: ET, India: IST)
- LinkedIn puts links in comments to avoid algorithm penalty on the main post
- No hashtags in the main post body — add them as a second comment if desired: #EngineeringLeadership #AITools #SoftwareEngineering #DeveloperProductivity
- The hook (METR study finding) is data-driven and non-obvious — this is what stops the scroll
- The "if you manage a team" line makes it shareable by engineering managers to their teams

---

## 5. Twitter/X Thread

### Tweet 1 (Hook)

```
A 2025 study found developers *believed* they were 20% faster with AI coding tools.

The actual measurement: 19% slower.

The problem isn't the tools. It's that we never learned how to use them properly.

I built an open-source system for that. Here's what it does:
```

### Tweet 2 (The Problem)

```
Most "AI for developers" content is vibes:

"Just write better prompts"
"Use AI for everything"
"10x engineer overnight"

None of it is installable. None of it measures where you actually are.

After studying practitioner reports from HN, Reddit, Addy Osmani, and Thoughtworks, I found 5 skills that separate productive AI users from frustrated ones.
```

### Tweet 3 (The 5 Skills)

```
The 5 dimensions that matter:

1. Spec discipline — write requirements before prompting
2. Context engineering — scope what AI sees
3. Review process — iterate on diffs, don't just run tests
4. Failure recovery — know when to revert and restart fresh
5. Tool orchestration — match tool to task, run in parallel

Each one is learnable. Each one compounds.
```

### Tweet 4 (What I Built)

```
So I built AI-Native Engineer — open source, free:

- 5 installable Claude Code skills (/audit, /test-gen, /refactor, /pr, /assess)
- Copy a folder. Run a slash command. Get value in 60 seconds.
- 3 drop-in configs that change how AI behaves at your skill level
- A scenario-based diagnostic (not self-reported yes/no)

No signup. No paywall.
```

### Tweet 5 (The Diagnostic)

```
The diagnostic is deliberately scenario-based.

Instead of "Do you write specs?" (everyone says yes), it asks:

"You need payment retry with exponential backoff. What's your first action?"

Your answer reveals your actual instincts, not your aspirational ones.

Maps you across 5 dimensions, not a single score.
```

### Tweet 6 (The Level Configs)

```
The part I'm most proud of: level configs.

Drop a CLAUDE.md file in your project and AI tools behave differently:

Level 1: Explains everything. One file at a time. Coaches you.
Level 2: Multi-file ops. Makes judgment calls. Flags them.
Level 3: Decomposes into parallel sub-tasks. Minimal confirmation.

Your tools grow with you.
```

### Tweet 7 (CTA)

```
Everything is here:

https://github.com/kamalkalwa/ai-native-engineer

Try the 90-second browser diagnostic:
https://kamalkalwa.github.io/ai-native-engineer/assess/

Or install a skill and run /audit on your codebase. You'll know in 5 minutes whether this is useful.

If it helps, star it so others find it too.
```

### Posting Notes
- Best time: Weekday 8-10am ET or 12-1pm ET
- Post Tweet 1, then reply-thread the rest immediately — do not space them out
- Pin the thread to your profile after posting
- No hashtags in the tweets themselves (algorithmic penalty on X for hashtags in 2026) — but you can add a reply at the end with: AI coding, developer tools, Claude Code, software engineering
- If the thread gets traction, quote-tweet Tweet 1 the next day with a specific result someone shared

---

## 6. Dev.to Article

### Title

```
I Built Installable AI Workflow Skills That Run in 60 Seconds — Here's How They Work
```

### Tags

```
ai, claudecode, productivity, opensource
```

### Body

````markdown
Every engineering team I talk to has adopted AI coding tools. Copilot, Cursor, Claude Code — the adoption wave is over.

But the *productivity* distribution is bimodal. Some engineers are genuinely faster. Others have the same tools and see marginal benefit.

After studying what separates the two groups — practitioner reports on HN, Reddit, Addy Osmani's writing, the Thoughtworks Technology Radar — I identified 5 specific skills that matter. And I turned them into an open-source system you can install and use in 60 seconds.

## The Problem with AI Coding Advice

Most of it isn't installable. "Write better prompts" doesn't give you anything to run. "Use AI for testing" doesn't tell you how to avoid the anti-patterns.

And most self-assessment is unreliable. A 2025 study by METR found developers believed they were 20% faster with AI coding tools, while the actual measurement showed 19% slower. Self-reported "yes I use AI well" is not a useful signal.

## What I Built

**AI-Native Engineer** is a GitHub repo with 5 installable Claude Code skills, 3 drop-in configuration files, a scenario-based workflow diagnostic, and a 4-week migration plan.

Repo: [github.com/kamalkalwa/ai-native-engineer](https://github.com/kamalkalwa/ai-native-engineer)

## Try It in 60 Seconds

```bash
# Clone the repo
git clone https://github.com/kamalkalwa/ai-native-engineer

# Copy one skill into your project
cp -r ai-native-engineer/skills/audit your-project/.claude/skills/audit

# Open Claude Code in your project
cd your-project && claude

# Run it
> /audit
```

The `/audit` skill runs a full codebase audit — security risks, dead code, performance issues, type safety gaps, error handling gaps, test coverage holes, dependency health. It outputs severity-rated tables with `file:line` references and suggested fixes.

That's it. One folder copy, one command.

## The 5 Skills

Each skill is a markdown file in `.claude/skills/` that Claude Code picks up as a slash command:

### `/audit` — Codebase Audit

Scans 8 categories:

1. Dead code (defined but never referenced)
2. Security risks (hardcoded secrets, injection vectors, missing validation)
3. Error handling gaps (functions that throw but aren't caught)
4. Performance concerns (N+1 queries, sync ops that should be async)
5. Type safety (any types, missing null checks)
6. Test coverage gaps (critical paths with no tests)
7. Dependency health (outdated versions, known CVEs)
8. Code duplication

Output format:

```
| Severity | File:Line | Issue | Suggested Fix |
```

### `/test-gen` — Test Generation with Guardrails

AI-generated tests have known anti-patterns. This skill has guardrails built in:

- Flags `toBeDefined()` / `toBeTruthy()` assertions as too weak
- Detects tests that assert against mock return values (testing the mock, not the code)
- Flags missing error path coverage
- Expects ~30% of assertions to need human rewriting — and tells you which ones

### `/refactor` — Multi-File Refactor

A guided workflow:

1. Inventory all instances of the pattern to change
2. Define the target pattern
3. Supervised application on first 3 files (you review each one)
4. Batch the remaining files
5. Verification pass

This prevents the "inconsistent refactor" anti-pattern where AI changes half the files one way and half another.

### `/pr` — Full PR Workflow

End-to-end:
- Implementation review
- Test generation
- Self-review (security, performance, edge cases)
- PR description with risk assessment and rollback plan
- Commit and push

### `/assess` — Workflow Diagnostic

Two layers:

**Layer 1 (automatic):** Scans your repo for signals — does CLAUDE.md exist? Are there spec files? AI co-authored commits? Test-to-source ratio? Multi-tool configs?

**Layer 2 (interactive):** 5 scenario questions. Not "do you write specs?" (everyone says yes). Instead: "You need payment retry with exponential backoff. What's your first action?" Your answer reveals your instincts.

Output is a per-dimension profile across 5 dimensions:

```
Your AI-Native Workflow Profile
═══════════════════════════════

Spec discipline:      Level 2  ← You write specs but don't reference existing patterns
Context engineering:  Level 1  ← You're pointing AI at the repo root
Review process:       Level 2  ← You review diffs but don't iterate with follow-ups
Failure recovery:     Level 3  ← You revert and restart — good instinct
Tool orchestration:   Level 1  ← Single tool for everything

Highest-ROI next step: Scope your AI's context window.
Install: cp levels/level-2-augmented.md CLAUDE.md
```

## The Level Configs

Three CLAUDE.md files that change how AI tools behave:

**Level 1 — Assisted:**
- Always explain before acting
- One file at a time
- Verify everything
- Coaches you on prompt quality

**Level 2 — Augmented:**
- Codebase-aware by default
- Multi-file operations
- Test generation built into workflow
- Makes judgment calls and flags them

**Level 3 — Native:**
- Orchestration mode
- Decomposes into parallel sub-tasks
- Minimal confirmation
- Self-reviews before presenting output

Drop one into your project root as `CLAUDE.md`. As you grow, swap the config.

## The Architecture

It's just markdown files. That's the whole point.

Each skill is a single SKILL.md file with frontmatter (name, description, allowed tools) and a structured prompt that encodes a complete workflow. Claude Code's skill system picks it up automatically.

```yaml
---
name: audit
description: Run a full codebase audit...
allowed-tools: Read, Grep, Glob, Bash(npx *), Bash(npm *)
---
```

No dependencies. No build step. No framework. Copy a folder, run the command.

## The Browser Diagnostic

If you don't use Claude Code, there's a browser version of the assessment:

[kamalkalwa.github.io/ai-native-engineer/assess](https://kamalkalwa.github.io/ai-native-engineer/assess/)

5 scenarios. 90 seconds. Same per-dimension profile. Gives you a copy-paste install command at the end.

## What This Isn't

- It's not a course. There's no video, no paywall, no signup.
- It doesn't make you a 10x engineer. The realistic number is 2-3x on output volume, and the quality ceiling is still your architecture sense and domain knowledge.
- The specific tools will change. The workflows and mental models are what last.

## Links

- **Repo:** [github.com/kamalkalwa/ai-native-engineer](https://github.com/kamalkalwa/ai-native-engineer)
- **Browser diagnostic:** [kamalkalwa.github.io/ai-native-engineer/assess](https://kamalkalwa.github.io/ai-native-engineer/assess/)
- **4-week migration plan:** [migration-plan/](https://github.com/kamalkalwa/ai-native-engineer/tree/main/migration-plan)

If you try `/audit` on your codebase, I'd genuinely like to hear how it goes.
````

### Posting Notes
- Best time: Tuesday-Thursday morning (Dev.to audience peaks midweek)
- Dev.to supports standard markdown — the code blocks will render correctly
- Use the canonical_url field if you also publish on your own blog
- Tags are limited to 4 on Dev.to — "ai, claudecode, productivity, opensource" covers the bases
- Consider adding a cover image: a screenshot of the `/audit` output or the browser diagnostic

---

## 7. Medium Article

### Title

```
The Engineering Job Market Split in Two. Here's the Playbook for the Right Side.
```

### Subtitle

```
A structured migration system for engineers transitioning to AI-native workflows — based on practitioner research, not hype.
```

### Body

```
There's a filter on every engineering job posting now, and it's not listed in the requirements.

It's not "5 years of React." It's not "distributed systems experience." It's this: does this person know how to ship with AI?

The teams that figured out AI-native workflows aren't hiring more engineers. They're shipping more with fewer. And the math is simple enough that any VP of Engineering can do it on a napkin:

Before: 10 engineers on traditional workflows = X output
After: 7 engineers on AI-native workflows = X output (or more)
Saved: 3 headcount of budget, reinvested in tooling

This isn't speculation. It's happening now, quietly, across the industry.

---

## The Measurement Problem

Here's what makes this transition tricky: engineers can't accurately self-assess.

A 2025 study by METR — one of the few rigorous measurements of AI coding tool productivity — found that experienced developers *believed* they were 20% faster when using AI tools. The actual measurement: 19% slower.

Read that again. Not "about the same." Not "slightly slower." The gap between perception and reality was nearly 40 percentage points.

This doesn't mean the tools don't work. It means we're in the awkward phase where the tools are powerful but our workflows haven't caught up. We're bolting AI onto 2022-era habits and wondering why the productivity graphs aren't hockey-sticking.

## What Actually Separates the Two Groups

I spent weeks going through practitioner reports from experienced engineers — Hacker News discussions, Reddit threads on r/ExperiencedDevs, Addy Osmani's writing on AI-assisted development, the Thoughtworks Technology Radar. Not vendor case studies. Not Twitter hot takes. Reports from people doing the work.

A pattern emerged. The engineers who are measurably more productive with AI tools share 5 specific skills:

**1. Spec Discipline**

This is the #1 differentiator in every practitioner report I read.

Productive engineers write a spec before they open an AI tool. Requirements, constraints, patterns to follow, acceptance criteria. Their AI output fits the codebase on the first try.

Less productive engineers type "add payment retry with exponential backoff" and spend 3-4 iterations fixing the generic output.

The rule of thumb: 40% of your time writing the spec, 20% waiting for AI output, 40% reviewing and iterating. If you're spending most of your time on review, your spec was too thin.

**2. Context Engineering**

"Removing irrelevant context improves performance dramatically." This insight keeps showing up across sources.

The instinct is to give AI as much context as possible. Dump the whole repo. Let it figure it out. This is wrong. AI performs better when you scope its context to the 3-4 relevant files.

The advanced pattern: use a research subagent to map the relevant code (in a separate context window), compress findings to ~50 lines, then start a fresh implementation session with only the compressed context and spec.

**3. Review Process**

Running tests and merging is not a review process. Neither is "LGTM."

The effective pattern: structural scan of each file, spot-check edge cases, feed corrections back as follow-up prompts, iterate until clean. Expect to rewrite about 30% of AI-generated test assertions — this is normal, not a failure.

**4. Failure Recovery**

When AI spirals — each fix introduces a new bug, the code is getting worse — the instinct is to give it more context and try again. This is the wrong instinct.

The right pattern: revert to the last known good state, start a completely fresh session, describe only the original error and the minimal relevant files. Sunk cost fallacy applies to AI sessions too.

**5. Tool Orchestration**

Not "which AI tool is best?" but using the right tool for each task. A 50-file logging migration belongs in a CLI-based agent. A visual React bug belongs in an editor with inline preview. An architecture decision belongs in a research conversation, not a code-generation session.

The Level 3 move: running independent tasks in parallel across multiple tools.

---

## The Four Levels

Based on these skills, there's a natural progression:

**Level 0: AI-Unaware.** You write all code manually. Your workflow hasn't changed since 2022.

**Level 1: AI-Assisted.** You have Copilot installed. You accept tab-completions. Maybe 20-30% faster on typing, but your workflow is fundamentally the same. This is where most engineers plateau — and it's the least valuable use of AI coding tools.

**Level 2: AI-Augmented.** You describe tasks to AI before coding. You handle multi-file changes. You generate tests and review every assertion. Your mental model shifted from "I code" to "I direct and review." Roughly 2x on tasks involving multi-file changes.

**Level 3: AI-Native.** You think in systems, not tasks. You decompose features into independent sub-tasks, define contracts, and integrate outputs. Multiple agents running in parallel. You don't measure yourself by lines of code — you measure by decisions made and outcomes shipped.

The key question at the end of each day: *"What did I do today that only a human could do?"* If the answer is "not much," you automated well. If the answer is "everything," you're not delegating enough.

---

## The Playbook

I built a system for this transition. Open source, free, no signup.

**It's a tool, not a guide.** The core idea: instead of reading about AI workflows, install them and use them.

**5 installable Claude Code skills:**
- `/audit` — full codebase audit with severity-rated file:line findings
- `/test-gen` — test generation with guardrails against common AI test anti-patterns
- `/refactor` — guided multi-file refactor that prevents inconsistent changes
- `/pr` — end-to-end PR workflow with risk assessment and rollback plan
- `/assess` — scenario-based diagnostic that maps you across the 5 dimensions

**3 drop-in configurations** that change how AI tools behave:
- Level 1 coaches you. Explains before acting. One file at a time.
- Level 2 works with you. Multi-file operations. Makes judgment calls and flags them.
- Level 3 gets out of your way. Decomposes tasks into parallel sub-tasks. Minimal confirmation.

**A 4-week migration plan** with daily exercises, specific prompts, tracking tables, and success criteria.

Copy one skill folder into your project, run the slash command, and see the value in 60 seconds. If it's useful, keep going. If not, you lost a minute.

---

## The Honest Caveats

AI output quality varies. You will get wrong code, hallucinated APIs, and subtly broken logic. The skill isn't trusting AI — it's reviewing AI output faster than you could write it yourself.

This doesn't make you a 10x engineer. The realistic number is 2-3x on output volume. Your quality ceiling is still your own architecture sense, debugging intuition, and domain knowledge.

The specific tools will change in 6 months. The workflows and mental models will last. Learn the thinking, not the buttons.

---

## The Bottom Line

The engineering job market has split into two tracks. One track has more candidates competing for fewer positions. The other has fewer candidates being recruited for growing positions.

The difference is a set of learnable skills. The transition takes about 4 weeks of deliberate practice. The tools range from free to $100/month.

The real cost is the time to change your habits — and that cost increases the longer you wait.

**The repo: [github.com/kamalkalwa/ai-native-engineer](https://github.com/kamalkalwa/ai-native-engineer)**

**The 90-second browser diagnostic: [kamalkalwa.github.io/ai-native-engineer/assess](https://kamalkalwa.github.io/ai-native-engineer/assess/)**

Start with the diagnostic. Install the level config it recommends. Try one skill on your real codebase. If it clicks, start Week 1.
```

### Posting Notes
- Best time: Tuesday or Wednesday morning
- Medium tags (up to 5): `Software Engineering`, `Artificial Intelligence`, `Developer Tools`, `Programming`, `Career Advice`
- Medium rewards longer reads — this length (1500-2000 words) performs well
- Add a simple cover image (a terminal screenshot or the diagnostic output would work)
- If you have a Medium publication in mind, submit there for broader distribution
- Consider adding the "Friend Link" when sharing so non-subscribers can read it
- No paywall unless you want monetization — for launch, free distribution is better for reach

---

## Posting Order (Recommended)

1. **Dev.to** (Tuesday morning) — establishes the technical depth, gets indexed
2. **Hacker News** (Tuesday 8-10am ET) — highest potential reach, time-sensitive
3. **Reddit r/programming** (same day, 1-2 hours after HN) — technical audience overlap
4. **Twitter/X** (same day, afternoon) — thread format, easy to share
5. **LinkedIn** (Wednesday morning) — professional audience, different demographic
6. **Reddit r/ExperiencedDevs** (Thursday) — space it out from r/programming to avoid appearing spammy
7. **Medium** (following week) — longer essay can reference early traction from other platforms

## Cross-Linking Strategy

- Dev.to and Medium articles can link to each other as "technical deep-dive" vs "narrative version"
- Twitter thread can link to the Dev.to article for "full technical writeup"
- LinkedIn post drives to the repo and diagnostic directly
- All posts should link to the GitHub repo and the browser diagnostic
