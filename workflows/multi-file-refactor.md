# Workflow: Multi-File Refactor with AI

**Time:** 2-4 hours for a typical service (10k-30k lines)
**Tools:** Claude Code (CLI) + Cursor Composer
**Level:** 2-3 (AI-Augmented to AI-Native)

---

## When to Use This

- You need to change a pattern that appears in 10+ files (error handling, logging, API calls, state management)
- You're migrating from one library/pattern to another across the codebase
- You're applying a new coding standard retroactively
- You need to rename or restructure a module that's imported everywhere

---

## Tool Selection: Claude Code vs. Cursor Composer

| Scenario | Use Claude Code | Use Cursor Composer |
|----------|----------------|-------------------|
| Changes span 10+ files | Yes | No — Composer struggles above ~15 files |
| You need to understand the full dependency graph first | Yes | No |
| Changes are scoped to 3-7 files you can see | No | Yes — inline diffing is faster to review |
| You need to run tests/builds between iterations | Yes — CLI integration | Possible but slower |
| The pattern is consistent (same change in every file) | Yes | Either works |
| The pattern requires per-file judgment calls | Either — but review carefully | Yes — easier to tweak inline |

**Rule of thumb:** Claude Code for discovery + bulk changes. Cursor Composer for surgical multi-file edits where you want visual diff review.

---

## Step 1: Identify All Instances of the Pattern

Run this in Claude Code:

```
Find every instance of the following pattern in this codebase:

Pattern: [describe the current pattern you want to change]

For each instance, output:
1. File path and line numbers
2. The current code (exact, not paraphrased)
3. What this specific instance does (context matters for the replacement)
4. Any edge cases or complications for THIS specific instance

Output as a numbered list. Do not skip any instances — I need a complete inventory.
```

### Example: Finding all try/catch error handling

```
Find every instance of try/catch error handling in this codebase.

For each instance, output:
1. File path and line numbers
2. The current code (exact, not paraphrased)
3. What this specific instance does (is it catching API errors, DB errors, validation errors, etc.)
4. Edge cases: Does the catch block do anything besides re-throwing? Does it modify state? Does it have cleanup logic in finally?

Output as a numbered list. Do not skip any instances.
```

---

## Step 2: Define the Target Pattern

Before asking AI to make changes, write a clear specification of the new pattern. This is your job, not the AI's. The AI applies the pattern; you design it.

```
Here is the new error handling pattern to apply across the codebase.

## The Result Type Pattern

Replace try/catch blocks with a Result type return pattern.

### Type Definition (already exists at src/types/result.ts):

type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

function Ok<T>(value: T): Result<T, never> {
  return { ok: true, value };
}

function Err<E>(error: E): Result<never, E> {
  return { ok: false, error };
}

### Transformation Rules:

BEFORE:
async function getUser(id: string): Promise<User> {
  try {
    const user = await db.users.findById(id);
    if (!user) throw new NotFoundError('User not found');
    return user;
  } catch (error) {
    logger.error('Failed to get user', { id, error });
    throw error;
  }
}

AFTER:
async function getUser(id: string): Promise<Result<User, AppError>> {
  const user = await db.users.findById(id).catch(
    (e) => Err(new AppError('DB_ERROR', 'Failed to get user', { id, cause: e }))
  );
  if (!user.ok) return user;
  if (!user.value) return Err(new AppError('NOT_FOUND', 'User not found', { id }));
  return Ok(user.value);
}

### Rules:
1. Every function that currently throws should return Result<T, AppError> instead
2. Callers must handle both branches (ok: true and ok: false)
3. DO NOT change functions in src/middleware/ — Express middleware needs to throw for the error handler
4. DO NOT change test files — we'll update tests separately
5. Import Ok, Err, Result from 'src/types/result'
6. Preserve all existing logging — move it into the Err() creation
7. If a function has a finally block with cleanup logic, keep the try/finally but remove the catch — use Result for the error path
```

---

## Step 3: Apply the Pattern

Run this in Claude Code:

```
Apply the new error handling pattern (defined above) to every instance
identified in the inventory.

Work through the list one at a time. For each file:
1. Show me the before and after
2. Flag any instance where the transformation isn't straightforward
   (has cleanup logic, catches specific error types, modifies external state in the catch block)
3. Update the callers of each changed function to handle the Result type

Do NOT change files in src/middleware/ or test files.

Start with file 1 from the inventory. After each file, wait for my confirmation
before proceeding to the next.
```

**Why "wait for confirmation":** For the first 2-3 files, review each transformation carefully. Once you see the AI is applying the pattern correctly and consistently, you can switch to batch mode:

```
The pattern looks correct. Apply it to the remaining files (4 through 17)
all at once. Flag any file where the transformation required a judgment
call different from the standard pattern.
```

---

## Step 4: Verification Workflow

After all changes are applied, run verification in this exact order:

### 4a. Type Checking

```bash
# TypeScript
npx tsc --noEmit

# If there are type errors, feed them back:
```

```
Here are the TypeScript errors after the refactor:
[paste errors]

Fix each one. Most will be callers that still expect the old return type.
Show me the fixes.
```

### 4b. Run Existing Tests

```bash
npm test 2>&1 | head -100
```

Tests will likely fail because return types changed. This is expected. Feed failures back:

```
These tests are failing after the refactor:
[paste failures]

Update each test to work with the new Result type pattern.
Rules:
- Test the ok: true path AND the ok: false path for each function
- Don't just add .value everywhere — verify the result is ok first
- Keep the existing test descriptions and structure
```

### 4c. Lint Check

```bash
npm run lint
```

### 4d. Build Check

```bash
npm run build
```

### 4e. Manual Smoke Test

Pick the 2-3 most critical paths through the application and test them manually. AI refactoring is good at mechanical transformation but can miss runtime behavior changes.

---

## Example Scenario: try/catch to Result Type Migration

### The codebase

Consider an Express + TypeScript API service with 14 service files, each averaging 5-8 functions with try/catch blocks. ~70 total instances to transform.

### Typical timeline

| Step | Time | What Happened |
|------|------|--------------|
| Pattern inventory | 8 min | Claude Code found 73 instances across 14 files. 4 were in middleware (excluded). 3 had finally blocks (flagged for manual review). |
| Define target pattern | 15 min | Wrote the Result type definition, transformation rules, and edge case handling. This was pure human work — designing the pattern. |
| Apply to first 3 files | 20 min | Reviewed each transformation carefully. Found one issue: Claude Code was wrapping async calls in an extra try/catch instead of using `.catch()`. Corrected the prompt. |
| Apply to remaining 11 files | 12 min | Batch mode. Claude Code flagged 2 files where the catch block modified external state (a cache invalidation). Handled those manually. |
| Fix TypeScript errors | 15 min | 34 type errors, all in callers that expected the old return type. Claude Code fixed all 34. |
| Fix test failures | 25 min | 19 test failures. Claude Code updated 16 automatically. 3 needed manual adjustment because they were testing that specific errors were thrown (now they test for `ok: false`). |
| Lint + build | 5 min | 2 lint issues (unused imports from removed try/catch). Auto-fixed. Build passed. |
| Manual smoke test | 15 min | Tested auth flow, payment flow, and user CRUD. All worked. |
| **Total** | **~2 hours** | 73 instances transformed across 14 files. Would have taken 2-3 days manually. |

### Files changed

```
 14 files changed, 847 insertions(+), 623 deletions(-)
 src/types/result.ts          | new file, 24 lines
 src/services/authService.ts  | 67 insertions, 49 deletions
 src/services/userService.ts  | 58 insertions, 42 deletions
 src/services/billingService.ts | 89 insertions, 61 deletions
 ... (11 more service files)
 src/routes/auth.ts           | 34 insertions, 21 deletions
 src/routes/users.ts          | 28 insertions, 18 deletions
 ... (6 more route files)
```

---

## Common Pitfalls

1. **Don't skip the pattern definition step.** If you tell the AI "refactor error handling to use Result types" without specifying exact transformation rules, each file will be refactored slightly differently. Consistency requires a written specification.

2. **Don't batch-apply before reviewing 2-3 examples.** The AI might misunderstand your pattern in a subtle way that compounds across 50 files. Catch it early.

3. **Don't forget to update callers.** Changing a function's return type from `Promise<User>` to `Promise<Result<User, AppError>>` breaks every caller. The AI handles this if you tell it to, but won't do it automatically.

4. **Don't refactor and add features simultaneously.** A refactor PR should contain only the refactor. If you mix in feature changes, the PR is unreviewable and rollback becomes dangerous.

5. **Don't skip the manual smoke test.** Type checking and tests catch most issues, but runtime behavior changes (especially around error propagation in async chains) can slip through. Spend 15 minutes clicking through critical paths.

6. **Watch for catch blocks with side effects.** A catch block that invalidates a cache, sends a metric, or modifies shared state is not just error handling — it's business logic. These need manual attention, not mechanical transformation.
