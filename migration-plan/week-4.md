# Week 4: Orchestration

**Goal:** Move from using one AI tool on one task to running multiple AI agents in parallel on decomposed tasks. By end of week, your daily workflow should feel fundamentally different from Week 0.

**Level transition:** 2 (AI-Augmented) to 3 (AI-Native)

---

## Day 1 (Monday): Task Decomposition

### The Key Skill

The gap between Level 2 and Level 3 is decomposition. A Level 2 engineer uses AI on one task at a time. A Level 3 engineer looks at a feature and splits it into 2-3 independent sub-tasks that AI can work on simultaneously.

### Exercise: Decompose a Feature

Pick a medium-to-large feature from your backlog — something that would normally take 1-2 days. Decompose it:

```
I need to build [feature description].

Break this down into sub-tasks that can be worked on independently
(no dependencies between them). For each sub-task:

1. What it produces (specific output: files, functions, tests)
2. What it needs as input (interfaces, types, specs — not other sub-task output)
3. Which AI tool is best for it
4. Estimated time

The sub-tasks should be independently implementable — I should be able
to run them in parallel and combine the outputs at the end.
```

### Exercise: Define Interfaces First

Before running any sub-tasks, define the contracts between them. This is the human job that enables parallel AI work.

```
Based on the sub-tasks above, define the TypeScript interfaces/types
that each sub-task will produce and consume.

These interfaces are the CONTRACTS. Each sub-task must implement its
side of the contract. When I combine the outputs, they should fit
together through these interfaces.

Output as a single types file that I'll create before starting any sub-task.
```

Create this types/interfaces file. It's the "glue" that lets parallel work combine cleanly.

### Success Criteria for Day 1
- [ ] Feature is decomposed into 2-3 independent sub-tasks
- [ ] Each sub-task has clear inputs, outputs, and tool assignment
- [ ] Interface contracts are defined and saved as a file

---

## Day 2 (Tuesday): Parallel Execution

### Morning: Run Sub-Tasks in Parallel

This is the day you run multiple AI agents simultaneously. Here's how:

**Terminal 1 — Claude Code (Service/Backend):**
```
Read the interface contracts in src/types/[feature].ts.

Implement sub-task 1: [description]

Constraints:
- Must implement the interfaces exactly as defined
- Follow existing patterns in [reference files]
- Include error handling for: [specific failure cases]
```

**Terminal 2 — Claude Code (Second instance or Cursor Composer):**
```
Read the interface contracts in src/types/[feature].ts.

Implement sub-task 2: [description]

Constraints:
- Must consume the interfaces exactly as defined
- Follow existing patterns in [reference files]
- Include error handling for: [specific failure cases]
```

**Claude Web or ChatGPT (Research/Documentation):**
```
I'm implementing [feature]. I need:
1. A migration guide for downstream consumers
2. An architecture decision record (ADR) for why we chose [approach]
3. Updated API documentation for the new endpoints

Context: [brief description of the system and the change]
```

### The Rhythm

While AI agents work:
1. Start the longest sub-task first
2. While it generates, start the second sub-task
3. While both generate, do the documentation/research task yourself or with Claude web
4. Review outputs as they come in. Don't wait for all to finish.
5. The first output to review is the one you're most worried about (highest risk)

### Afternoon: Review and Iterate

For each sub-task output:
1. Does it implement the interface contracts correctly?
2. Does it follow codebase patterns?
3. Are there edge cases missing?

Fix issues in each sub-task independently. Don't combine yet.

### Success Criteria for Day 2
- [ ] Ran 2+ AI tasks in parallel
- [ ] Each sub-task output implements the interface contracts
- [ ] Each sub-task is independently reviewed and corrected
- [ ] You experienced the time savings of parallel execution

---

## Day 3 (Wednesday): Integration and Testing

### Morning: Combine the Sub-Tasks

This is the human-judgment step. The AI generated the pieces; you connect them.

```
I have implementations of [sub-task 1] and [sub-task 2] that share
interfaces defined in src/types/[feature].ts.

Review both implementations and:
1. Identify any conflicts or inconsistencies between them
2. Identify any gaps — things that neither sub-task handles
   (usually: error propagation between components, edge cases
   at the boundary)
3. Show me the integration code needed to connect them
4. Flag anything that needs manual attention
```

### Exercise: Write Integration Tests

The unit tests from each sub-task test components in isolation. Integration tests verify they work together:

```
Generate integration tests for the [feature] that test the full flow:

1. [End-to-end happy path: user action → all layers → final result]
2. [Failure in sub-task 1 component → how sub-task 2 component handles it]
3. [Failure in sub-task 2 component → how the system recovers]
4. [Concurrent access scenario, if applicable]

These tests should NOT mock internal components — they should test
the real integration between the pieces. Only mock external boundaries
(database, external APIs).
```

### Afternoon: Full Verification

```bash
# Type check
npx tsc --noEmit

# All tests (unit + integration)
npm test

# Build
npm run build
```

### Success Criteria for Day 3
- [ ] Sub-tasks are integrated into a working feature
- [ ] Integration tests verify the components work together
- [ ] All tests pass (unit + integration)
- [ ] Build succeeds

---

## Day 4 (Thursday): Build One Automation

### The Goal

Level 3 engineers don't just use AI tools — they build workflows that compose them. Today, build one automation that uses AI programmatically.

### Exercise: Pick and Build an Automation

Choose one that fits your daily work:

**Option A: Pre-commit AI Review**

Create a git hook that runs AI review on staged changes before commit:

```bash
#!/bin/bash
# .git/hooks/pre-commit (or use husky/lefthook)

DIFF=$(git diff --cached)

if [ -z "$DIFF" ]; then
  exit 0
fi

echo "Running AI pre-commit review..."

REVIEW=$(echo "$DIFF" | claude -p "Review this diff for:
1. Security issues (hardcoded secrets, SQL injection, XSS)
2. Obvious bugs (null derefs, off-by-one, missing await)
3. Performance issues (N+1 queries, missing pagination)

If no issues found, respond with only: LGTM
If issues found, list them with severity and file:line references.
Respond concisely — this runs on every commit.")

if echo "$REVIEW" | grep -q "LGTM"; then
  echo "AI review: LGTM"
  exit 0
else
  echo "AI review found issues:"
  echo "$REVIEW"
  echo ""
  echo "Commit anyway? (y/n)"
  read -r response
  if [ "$response" = "y" ]; then
    exit 0
  else
    exit 1
  fi
fi
```

**Option B: PR Description Generator Script**

Create a script that generates a PR description from the current branch:

```bash
#!/bin/bash
# scripts/generate-pr-description.sh

BASE_BRANCH=${1:-main}
DIFF=$(git diff "$BASE_BRANCH"...HEAD)
COMMITS=$(git log "$BASE_BRANCH"..HEAD --oneline)

echo "$DIFF" | claude -p "Generate a PR description for these changes.

Commits:
$COMMITS

Format:
## What Changed
[bullet list]

## Why
[1-2 sentences]

## Risk Assessment
| Risk | Likelihood | Impact | Mitigation |
[table rows]

## Rollback Plan
[specific steps]

## Testing
[checklist]"
```

**Option C: Test Coverage Gap Finder**

Create a script that identifies untested code paths:

```bash
#!/bin/bash
# scripts/find-test-gaps.sh

MODULE=$1

if [ -z "$MODULE" ]; then
  echo "Usage: ./find-test-gaps.sh src/services/userService.ts"
  exit 1
fi

claude -p "Read $MODULE and its corresponding test file.

For each public function in the module:
1. List all code paths (branches, error conditions, edge cases)
2. Check which paths have test coverage
3. List the UNTESTED paths

Output format:
| Function | Total Paths | Tested | Untested | Missing Test Description |

Then generate the missing tests."
```

### Exercise: Use Your Automation

After building it, use it 3+ times during the day. Refine it based on actual usage:
- Is the output format useful?
- Is it fast enough to not disrupt your flow?
- Does it catch real issues?

### Success Criteria for Day 4
- [ ] Built one automation that uses AI programmatically
- [ ] Used it at least 3 times on real work
- [ ] Refined it based on actual usage

---

## Day 5 (Friday): Workflow Audit and Final Metrics

### Morning: The Daily Workflow Audit

Spend the morning doing your normal work, but track every action in this table:

| Time | Action | Tool Used | Could AI Have Done This? | Did I Use AI? |
|------|--------|-----------|-------------------------|---------------|
| 9:00 | Read ticket #234 | Jira | No | N/A |
| 9:05 | Understand codebase context | Claude Code | Yes | Yes |
| 9:15 | Design approach | Whiteboard | No (architecture decision) | N/A |
| 9:25 | Implement feature | Claude Code | Yes | Yes |
| ... | ... | ... | ... | ... |

At the end of the morning, calculate:
- Total tasks: ___
- Tasks where AI could help: ___
- Tasks where you used AI: ___
- Delegation rate: `AI_used / AI_could_help × 100` = ___%

**Target:** >80% delegation rate. If lower, identify what you're doing manually that AI could handle.

### Afternoon: Final Assessment and Week 4 Metrics

**Week 4 Specific Metrics:**
1. Feature decomposed into parallel sub-tasks: ___
2. Sub-tasks run in parallel: ___
3. Time to complete feature (with parallel execution): ___ hours
4. Estimated time without parallel execution: ___ hours
5. Time savings from parallelization: ___%
6. Automation built: [description]
7. Times automation was used on Day 4-5: ___

**Overall 4-Week Comparison:**

| Metric | Week 1 | Week 2 | Week 3 | Week 4 |
|--------|--------|--------|--------|--------|
| Tasks completed with AI | ___ | ___ | ___ | ___ |
| Avg time per task (min) | ___ | ___ | ___ | ___ |
| Manual coding reverts | ___ | ___ | ___ | ___ |
| Files changed per AI session | ___ | ___ | ___ | ___ |
| Prompt quality (first-try usable %) | ___ | ___ | ___ | ___ |
| Test rewrite rate | N/A | ___% | ___% | ___% |

**Re-take the Workflow Diagnostic:**

Use the [interactive diagnostic](https://kamalkalwa.github.io/ai-native-engineer/assess/) or the `/assess` skill. Compare your per-dimension profile to your Week 0 results.

| Dimension | Week 0 | Week 4 |
|-----------|--------|--------|
| Spec discipline | Level ___ | Level ___ |
| Context engineering | Level ___ | Level ___ |
| Review process | Level ___ | Level ___ |
| Failure recovery | Level ___ | Level ___ |
| Tool orchestration | Level ___ | Level ___ |

### The Honest Check

Ask yourself: **"Does my daily workflow feel fundamentally different from 4 weeks ago?"**

- **Yes →** You've made the transition. Keep practicing and your speed will continue to improve. Focus on building more automations and teaching the workflow to colleagues.
- **Somewhat →** You've improved but haven't fully committed. Go back to Week 2 and focus on making codebase-aware prompts automatic. The gap is usually in prompt quality.
- **No →** Something blocked your adoption. Common reasons: your codebase is too locked down for AI tools, your tasks are too small to benefit from AI workflows, or you didn't actually do the daily exercises. Identify the blocker and address it specifically.

### Success Criteria for Day 5
- [ ] Daily workflow audit completed with delegation rate calculated
- [ ] All 4-week metrics recorded and compared
- [ ] Self-assessment retaken with before/after scores
- [ ] Honest evaluation of whether your workflow has changed

---

## Week 4 Success Criteria (Overall)

You have completed the migration if:

1. **You can decompose features into parallel AI tasks.** Given a feature, you can split it into 2-3 independent sub-tasks, define interface contracts, run them in parallel, and integrate the results. This is the core Level 3 skill.

2. **You run multiple AI agents simultaneously.** Your daily workflow includes at least one instance of parallel AI execution. You don't wait for one task to finish before starting the next.

3. **You've built at least one automation.** A script, hook, or pipeline step that uses AI programmatically. You understand that AI is not just a chat interface — it's a tool you can invoke from code.

4. **Your self-assessment score improved by 3+ points.** If you started at 2/10 and you're at 5/10, you've moved from AI-Curious to AI-Assisted. If you started at 5/10 and you're at 8/10, you've moved from AI-Assisted to AI-Native.

5. **Your daily workflow feels different.** Not incrementally different — structurally different. You think in systems, delegate by default, review instead of write, and measure your output in decisions and outcomes, not lines of code.

---

## What Comes After Week 4

### Month 2: Depth
- Master harness prompts for your specific domain
- Build 3-5 automations that cover your most common workflows
- Start contributing to or evaluating AI developer tools
- Teach the workflow to one colleague

### Month 3: Leadership
- Propose AI-native workflow changes for your team
- Build team-specific prompt libraries and automation scripts
- Measure team-level metrics (PR velocity, deployment frequency)
- Identify which team processes can be restructured for AI-native workflows

### Ongoing
- The tools change monthly. Re-evaluate your toolkit every 4-6 weeks.
- The mental models last. Decomposition, delegation, review, and orchestration are permanent skills.
- Your competitive advantage isn't the tools — it's the workflow thinking. Tools are commodities. The ability to orchestrate them is the skill that separates levels.

---

## Common Pitfalls for Week 4

1. **"I can't run tasks in parallel — they all depend on each other."** Most tasks have more independence than you think. The trick is defining the interface contracts up front. If Task A produces data that Task B consumes, define the data shape first, then both tasks can work against that contract simultaneously.

2. **"My automation is too brittle — the AI output format is unpredictable."** Add explicit output format instructions to your prompts: "Respond with ONLY a JSON object with these keys..." or "Respond with ONLY 'LGTM' or a bullet list of issues." Constrained output is more reliable for automation.

3. **"I don't know if I'm actually faster or just feel busier."** Measure. Compare: how many PRs did you ship in Week 4 vs. a typical pre-migration week? How many hours per PR? Feelings are unreliable; numbers are not.

4. **"My team doesn't use AI tools, so I can't change the workflow."** You can change YOUR workflow without team buy-in. Run your own pre-commit hooks. Generate your own PR descriptions. When your metrics show improvement, the team will notice. Lead by example, not by mandate.

5. **"Week 4 feels too advanced — I'm not ready."** If you completed Weeks 1-3 honestly, you're ready. The jump from "one task at a time" to "parallel tasks" feels big conceptually but is small mechanically — it's just opening two terminal windows instead of one. Start with 2 parallel tasks. Expand from there.
