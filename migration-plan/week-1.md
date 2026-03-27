# Week 1: Install and Commit

**Goal:** Make AI your default starting point for every coding task. By the end of this week, opening an AI tool before opening a file should feel natural.

**Level transition:** 0 (AI-Unaware) to 1 (AI-Assisted)

---

## Day 1 (Monday): Setup and First Contact

### Morning: Install Everything (~30-60 min including account setup)

> **Cost:** Cursor has a free tier (limited completions). Claude Code requires either a Max subscription ($100/mo) or API credits (pay-per-use — typical cost: a few dollars/day for active use). Copilot has a free tier. **Total to start: $0 with free tiers + API credits.** If you need to try before paying, use Cursor free + Copilot free for Week 1, then add Claude Code with API credits for Week 2.

1. **Install Cursor** — Download from cursor.com. Open your primary project in it. Do not configure anything fancy yet — defaults are fine. Account creation + download + first indexing takes ~10-15 min.

2. **Install Claude Code** — Run in your terminal:
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```
   Navigate to your main project directory and run `claude` to verify it works. Let it index the codebase. First-time setup (authentication, indexing) takes ~10-20 min depending on codebase size.

3. **Install GitHub Copilot** — If you don't already have it, install the VS Code/Cursor extension. Free tier is sufficient for Week 1.

### Afternoon: Complete One Real Task with AI

Pick the smallest task on your board — a bug fix, a small feature, a config change. Something you'd normally finish in 30-60 minutes.

**Exercise:** Complete this task using Claude Code.

```
Read the codebase context for [area of the task].

Here's what I need to do: [paste ticket or describe the task].

What files need to change? Show me the approach before writing code.
```

Review the approach. Then:

```
Implement this approach. Show me the changes.
```

**What you're learning:** Not whether AI produces perfect code (it won't), but the rhythm of describe-review-iterate. The point is to experience the workflow, not to optimize it yet.

### Success Criteria for Day 1
- [ ] Cursor, Claude Code, and Copilot are installed and functional
- [ ] You completed one real task using AI assistance
- [ ] You noticed at least one thing AI did well and one thing it got wrong

---

## Day 2 (Tuesday): AI-First for Every Task

### The Rule
Today, for EVERY task — no matter how small — your first action is describing it to an AI before writing code. Even if you know exactly what to do. Even if the AI suggestion is worse than what you'd write. The goal is building the habit, not optimizing the output.

### Exercise 1: Bug Fix with AI

Take a bug from your tracker. Use this prompt:

```
Here's a bug report:
[paste the bug description, error message, or reproduction steps]

Read the relevant code and explain:
1. What's causing this bug
2. Where the fix should go (file and function)
3. The fix itself, with the diff
```

**Track:** How long did AI-assisted debugging take vs. your estimate for doing it manually?

### Exercise 2: Code Review with AI

If you have a PR to review today, try this before reading the code yourself:

```
Review this diff for:
1. Bugs or logic errors
2. Missing edge cases
3. Anything that could break existing functionality

[paste the diff or point Claude Code to the branch]
```

Read the AI review first, then do your own review. Compare what AI caught vs. what you caught.

### Success Criteria for Day 2
- [ ] Every task today started with an AI prompt, not with opening a file
- [ ] You completed at least one bug fix with AI assistance
- [ ] You have a rough sense of where AI helps and where it wastes time

---

## Day 3 (Wednesday): Learning Prompt Quality

### The Experiment

Take the same task and write two prompts: a bad one and a good one. Compare the outputs.

**Exercise: Bad Prompt vs. Good Prompt**

Bad prompt:
```
add input validation to the signup form
```

Good prompt:
```
Add input validation to the signup form in src/pages/Signup.tsx.

Current state: The form submits directly to the API with no client-side validation.

Validation rules:
- Email: must be valid format, must not be a disposable email domain
- Password: minimum 8 characters, at least one number and one special character
- Name: 1-100 characters, no HTML tags

Validation library: We use zod throughout the codebase (see src/schemas/ for examples).

Show validation errors inline below each field, using our existing
ErrorMessage component from src/components/ErrorMessage.tsx.

Do not change the API call or the form submission handler — only add
client-side validation before the submit.
```

Run both prompts. Compare:
- Which output required less editing?
- Which output matched your codebase patterns?
- Which output handled edge cases?

### Key Principle

The quality gap between these two prompts is the difference between "AI isn't useful" and "AI writes 80% of my code." Every minute spent on prompt clarity saves 5 minutes on editing AI output.

### Exercise: Write 3 Good Prompts

Pick 3 tasks from your backlog. For each one, write a detailed prompt (like the "good" example above) but don't run them yet. Save them. You'll use them on Day 4 and Day 5.

### Success Criteria for Day 3
- [ ] You ran the bad vs. good prompt experiment and observed the quality difference
- [ ] You wrote 3 detailed prompts for upcoming tasks
- [ ] You can articulate what makes a good prompt (context, constraints, examples, output format)

---

## Day 4 (Thursday): Speed and Iteration

### The Goal

Today is about speed. Use the prompts you wrote yesterday and complete the tasks. Focus on the iteration loop: prompt, review, correct, prompt again.

### Exercise: Complete 2-3 Tasks with AI

Use your saved prompts from Day 3. For each task:

1. Run the prompt
2. Review the output in under 5 minutes. Don't read every line — scan for structural correctness, then spot-check details.
3. If something's wrong, don't rewrite it manually. Describe the issue to AI:

```
In the output above, the email validation is using a simple regex
instead of the zod schema approach. Look at src/schemas/userSchema.ts
for how we do validation. Rewrite the validation to follow that pattern.
```

4. Iterate until the output is correct, then apply the changes.

### Track Your Time

For each task, record:
- How long the AI-assisted approach took
- Your estimate for how long it would have taken manually
- How many iteration rounds were needed

| Task | AI Time | Manual Estimate | Iterations |
|------|---------|----------------|------------|
| Task 1 | __ min | __ min | __ |
| Task 2 | __ min | __ min | __ |
| Task 3 | __ min | __ min | __ |

**Expected result:** Day 4 is often slower than manual for some tasks. This is normal. You're building a skill. By Week 2, the speed advantage becomes consistent.

### Success Criteria for Day 4
- [ ] You completed 2-3 real tasks using AI
- [ ] You practiced iteration (correcting AI output with follow-up prompts instead of manual editing)
- [ ] You have time data comparing AI-assisted vs. estimated manual effort

---

## Day 5 (Friday): Reflection and Baseline

### Morning: Complete Any Remaining Tasks with AI

Use the same workflow. By now, describe-review-iterate should feel less awkward and more like a workflow.

### Afternoon: Measure Your Week 1 Baseline

Answer these questions honestly. Write the answers down — you'll compare against Week 4.

**Quantitative:**
1. How many tasks did you complete this week with AI assistance? ___
2. How many times did you write code without consulting AI first? ___
3. What percentage of AI output did you use as-is vs. rewrite? ___% as-is / ___% rewritten
4. Average time per task (AI-assisted): ___ minutes
5. Average iterations per task: ___

**Qualitative:**
6. What types of tasks was AI most helpful for? ___
7. What types of tasks was AI least helpful for? ___
8. What's the biggest time sink: writing the prompt, reviewing the output, or iterating on corrections? ___
9. Did AI ever produce something better than what you would have written? ___
10. What's one thing you'd do differently next week? ___

### Exercise: Identify Your Week 2 Focus

Based on your Week 1 experience, pick the specific area where AI had the most potential but you didn't fully tap into it. Common answers:

- "I kept reverting to manual coding when the AI output wasn't perfect" → Week 2 focus: iteration, not rewriting
- "My prompts were still too vague" → Week 2 focus: prompt engineering, always include context and constraints
- "I only used AI for small tasks" → Week 2 focus: multi-file tasks with Claude Code
- "I didn't trust the output enough to ship it" → Week 2 focus: verification workflows (tests, type checking)

### Success Criteria for Day 5
- [ ] Week 1 baseline metrics are recorded
- [ ] You identified your specific improvement area for Week 2
- [ ] You used AI for every task today (the habit should be forming)

---

## Week 1 Success Criteria (Overall)

You are ready for Week 2 if:

1. **Tool setup is complete.** Cursor, Claude Code, and Copilot are installed, configured, and used daily.

2. **AI-first is becoming a habit.** By Friday, your instinct when starting a task should be to describe it to AI, not to open a file. If you're still defaulting to manual coding, repeat Week 1.

3. **You can write a good prompt.** You understand that context, constraints, and specificity determine output quality. You don't just say "write a function" — you specify the function's contract, the codebase patterns it should follow, and the edge cases to handle.

4. **You've experienced the iteration loop.** You've used follow-up prompts to correct AI output at least 5 times this week. You know the difference between "rewrite it yourself" and "tell the AI what's wrong."

5. **You have baseline metrics.** You recorded your Week 1 numbers. These are your comparison points for measuring improvement.

---

## Common Pitfalls for Week 1

1. **"I'll try AI when I have a good task for it."** No. Use it for every task, even ones where it's slower. You're building a habit, not optimizing for output this week.

2. **"The AI output was wrong, so I rewrote it manually."** Stop. Tell the AI what was wrong. Iteration is the skill you're building. Manual rewriting is the habit you're breaking.

3. **"I don't have time to learn new tools this week."** You're not learning new tools — you're doing your existing tasks with an assistant. The time investment is 10-15 minutes on Day 1 for setup. Everything else is your normal work, done differently.

4. **"The AI doesn't understand our codebase."** Claude Code reads your repo. If the output doesn't match your patterns, your prompt is missing context. Add: "Follow the patterns in [specific file]. Use our existing [specific utility/component/pattern]."

5. **"I feel slower, not faster."** Expected. Days 1-3 are often slower. The crossover usually happens around Day 4-5. If you're still slower at end of Week 1, your prompts need work — that's your Week 2 focus.
