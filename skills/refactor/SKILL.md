---
name: refactor
description: Guided multi-file refactor — inventory all instances, define target pattern, supervised application, batch the rest, verify. Use when changing a pattern across 5+ files.
argument-hint: [pattern-description]
allowed-tools: Read, Grep, Glob, Edit, Write, Bash(npx *), Bash(npm *)
---

# Multi-File Refactor

Guide the user through a structured, safe multi-file refactor. Follow these phases in order. Do not skip phases.

## Phase 1: Inventory

Ask the user: "What pattern do you want to change? Describe the current pattern and I'll find every instance."

Then find every instance of the pattern in the codebase. For each instance, output:
1. File path and line numbers
2. The exact current code (not paraphrased)
3. What this specific instance does (context matters for the replacement)
4. Complications: edge cases, side effects, cleanup logic, or anything non-standard

Output as a numbered list. Be exhaustive — the user needs a complete inventory before changing anything.

## Phase 2: Target Pattern

Ask the user to define the target pattern. If they give a vague instruction ("modernize", "clean up", "improve"), push back:

"Vague prompts produce inconsistent results across files. Can you specify:
1. A before/after example
2. Transformation rules (mechanical steps I can follow)
3. Exceptions — files or patterns to NOT change"

If the user provides a clear spec, confirm it back to them before proceeding.

## Phase 3: Supervised Application (First 3 Files)

Apply the pattern to the first 3 files one at a time. For each file:
- Show before and after
- Flag anything that doesn't fit the standard pattern
- Wait for the user's confirmation before proceeding to the next file

After 3 files, ask: "Pattern looks correct? I can batch the remaining [N] files, or continue one-by-one."

## Phase 4: Batch Application

Apply to remaining files. For each file, flag any instance that required a non-standard transformation.

## Phase 5: Verification

Run in this order:
1. Type checking (`npx tsc --noEmit` or equivalent)
2. Tests (`npm test` or equivalent)
3. Lint (`npm run lint` or equivalent)

Feed any errors back and fix them. Then ask:

"All automated checks pass. I recommend manually testing your 2-3 most critical user flows before merging. Which flows should we verify?"
