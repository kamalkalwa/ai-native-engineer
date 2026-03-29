# Workflow: AI-Generated Tests

**Time:** ~30 minutes per module
**Tools:** Claude Code (CLI) or Cursor Composer
**Level:** 2 (AI-Augmented)

---

## When to Use This

- You need to add tests to existing code that has low or no coverage
- You've built a new feature and need to write the test suite
- You're refactoring and need to lock down behavior before making changes
- You want to increase coverage on a specific module without spending a full day on it

---

## Step 1: The Prompt That Produces Behavior-Based Tests

This is the single most important part. The default AI test output tests implementation details. This prompt steers it toward behavior.

```
Generate tests for [file path or module name].

Rules:
1. Test BEHAVIOR, not implementation. Each test name should complete the
   sentence "it should ___" with something a product manager would understand.

   GOOD: "it should return an error when the email is already registered"
   BAD:  "it should call userRepository.findByEmail with the correct argument"

2. Test the PUBLIC interface only. Do not test private methods, internal
   state, or how the function achieves its result. Test WHAT it returns
   or WHAT side effects it produces.

3. For each function, test these categories:
   - Happy path (valid input → expected output)
   - Invalid input (each type of bad input → appropriate error/response)
   - Edge cases (empty arrays, null values, boundary numbers, unicode strings)
   - Error conditions (database down, API timeout, network failure)

4. Mock EXTERNAL dependencies (database, APIs, file system), not internal
   modules. If function A calls function B in the same service, do NOT
   mock function B — let it run.

5. Use descriptive test data, not random strings.
   GOOD: const email = "already-registered@company.com"
   BAD:  const email = "test@test.com"

6. Each test should be independent. No shared mutable state between tests.
   Setup goes in beforeEach, not in shared variables that tests mutate.

7. Assert on the COMPLETE return value when practical, not just one field.
   GOOD: expect(result).toEqual({ id: "user-1", email: "new@co.com", role: "member" })
   BAD:  expect(result.id).toBeDefined()

8. For async error cases, test that the error contains useful information,
   not just that it threw.
   GOOD: expect(error.message).toContain("User with email already-registered@company.com already exists")
   BAD:  expect(fn).toThrow()

Output the complete test file with all imports and setup.
Testing framework: [jest/vitest/pytest/etc.]
```

---

## Step 2: The 30% Rewrite Rule

After the AI generates tests, expect to rewrite approximately 30% of the assertions. Here's what to look for:

### Assertions to Rewrite

| AI Wrote This | Problem | Rewrite To |
|---------------|---------|------------|
| `expect(mockDb.save).toHaveBeenCalledWith(userData)` | Tests implementation (how data is saved), not behavior (what happens after saving) | `expect(result).toEqual({ id: expect.any(String), ...userData })` |
| `expect(mockDb.save).toHaveBeenCalledTimes(1)` | Tests call count — breaks if implementation adds caching or batching | Remove entirely, or test the actual outcome |
| `expect(result).toMatchSnapshot()` | Lazy assertion — changes to any field break the test with no information about what matters | `expect(result).toEqual({ specificField: expectedValue })` |
| `expect(result).toBeDefined()` | Asserts existence, not correctness — almost never catches real bugs | `expect(result).toEqual(expectedFullObject)` |
| `expect(result).toBeTruthy()` | `0`, `""`, and `[]` are all falsy — this assertion is ambiguous | `expect(result).toBe(true)` or assert the specific value |
| `expect(fn).toHaveBeenCalled()` | Only checks if called, not with what arguments or what it did | Test the outcome of the call, not that it happened |
| `expect(internalState.cache.size).toBe(1)` | Tests internal state that consumers don't care about | Test the external behavior that the cache enables (e.g., second call is faster or returns same reference) |

### Assertions to Keep

| AI Wrote This | Why It's Good |
|---------------|---------------|
| `expect(result.statusCode).toBe(401)` | Tests observable behavior — what the caller receives |
| `expect(result.body.error).toBe("Invalid credentials")` | Tests the error message the user sees |
| `expect(auditLog.entries).toContainEqual({ action: "LOGIN_FAILED", userId: "user-1" })` | Tests a meaningful side effect — audit logging is a business requirement |
| `expect(emailService.send).toHaveBeenCalledWith(expect.objectContaining({ to: "user@co.com", template: "welcome" }))` | Mocking an external service AND verifying the contract is correct — this is appropriate |

---

## Step 3: Common AI Test Anti-Patterns

Watch for these in every AI-generated test file. They appear consistently across models.

### Anti-Pattern 1: Testing Mock Call Counts

```typescript
// BAD — AI loves this pattern
it('should call the database once', () => {
  await createUser(userData);
  expect(mockDb.insert).toHaveBeenCalledTimes(1);
});
```

**Why it's bad:** If you later add a cache write or an audit log insert, this test breaks even though behavior is correct. Call counts are implementation details.

**When call count testing IS valid:** Rate limiting, retry logic, or circuit breakers where the count is the behavior being tested.

### Anti-Pattern 2: Testing Internal State

```typescript
// BAD — AI accesses private internals
it('should update the internal cache', () => {
  await service.getUser("user-1");
  expect((service as any)._cache.has("user-1")).toBe(true);
});
```

**Why it's bad:** Internal cache is an optimization, not a behavior. Test that the second call returns the same data, or that it's faster — not that the cache key exists.

### Anti-Pattern 3: Snapshot Overuse

```typescript
// BAD — Snapshot tests for dynamic data
it('should return user data', () => {
  const result = await getUser("user-1");
  expect(result).toMatchSnapshot();
});
```

**Why it's bad:** Snapshot tests assert on everything, including fields you don't care about (timestamps, generated IDs). Any change to any field fails the test with a diff that doesn't tell you if the change matters.

**When snapshots ARE valid:** UI component rendering where you want to catch unintended visual changes. API response shapes as a contract test. Never for objects with dynamic fields.

### Anti-Pattern 4: Testing the Mock, Not the Code

```typescript
// BAD — This test passes even if createUser is deleted
it('should create a user', () => {
  mockDb.insert.mockResolvedValue({ id: "user-1", name: "Test User" });
  const result = await createUser({ name: "Test User" });
  expect(result).toEqual({ id: "user-1", name: "Test User" });
});
```

**Why it's bad:** The test just verifies that the mock returns what you told it to return. If `createUser` doesn't validate input, doesn't hash passwords, doesn't check for duplicates — this test still passes.

**Fix:** Assert on behavior that proves the function actually did its job:

```typescript
it('should hash the password before storing', () => {
  await createUser({ name: "Test User", password: "plaintext123" });
  const savedData = mockDb.insert.mock.calls[0][0];
  expect(savedData.password).not.toBe("plaintext123");
  expect(savedData.password).toMatch(/^\$2[aby]\$/); // bcrypt hash pattern
});
```

### Anti-Pattern 5: No Error Path Tests

AI tends to generate 8 happy-path tests and 1 generic error test. The error paths are where production bugs live.

**Fix:** For every function, explicitly ask: "What are the 3-5 ways this function can fail?" and ensure each has a test.

---

## Example Scenario: Generating Tests for a Payment Service

### The module: `src/services/paymentService.ts`

```typescript
export class PaymentService {
  constructor(
    private stripe: Stripe,
    private db: Database,
    private emailService: EmailService
  ) {}

  async chargeCustomer(customerId: string, amount: number, currency: string): Promise<ChargeResult> { ... }
  async refund(chargeId: string, reason: string): Promise<RefundResult> { ... }
  async getPaymentHistory(customerId: string, options: PaginationOptions): Promise<PaymentHistory> { ... }
}
```

### The prompt

```
Generate tests for src/services/paymentService.ts.

Rules:
1. Test BEHAVIOR, not implementation. Test what the function returns and
   what side effects it produces, not how it calls internal methods.
2. Test the PUBLIC interface only (chargeCustomer, refund, getPaymentHistory).
3. For each function test: happy path, invalid input, edge cases, error conditions.
4. Mock Stripe, Database, and EmailService — these are external dependencies.
5. Use descriptive test data (real-looking amounts, currency codes, customer IDs).
6. Assert on complete return values, not just existence.
7. For error cases, verify the error contains useful debugging information.

Testing framework: vitest
```

### What to expect from AI output

For a module like this, AI will typically generate 20-30 tests. Here's the pattern you'll see:

**Keep as-is (~70%):**
- Happy-path tests that assert on return values
- Invalid input tests (negative amount, empty currency, nonexistent customer)
- Pagination edge cases (empty results, last page)

**Rewrite assertions (~20-30%):**
- `toHaveBeenCalledTimes(1)` → assert the returned object has the correct values instead
- `toBeDefined()` → `toEqual({ id: expect.any(String), amount: 4999, currency: "usd", status: "succeeded" })`
- Snapshot assertions → explicit field assertions
- `toHaveBeenCalled()` → `toHaveBeenCalledWith(expect.objectContaining({ to: "...", template: "..." }))`
- `toBeTruthy()` → `toContain("Insufficient funds for customer cus_123")`

**Delete (~5-10%):**
- Tests that only verify mock call parameters (testing implementation, not behavior)
- Tests that check side-effect ordering rather than outcomes

**Add manually (~2-3 tests):**
- Idempotency: same request retried = same outcome
- Currency/precision handling: edge cases AI rarely thinks of
- Concurrency: two simultaneous requests don't cause double-processing

### Typical time

```
Prompt + generation:     5-10 min
Review + rewrite:        15-20 min
Total:                   ~30 min per module
Equivalent manual effort: 2-3 hours
```

---

## Workflow Summary

```
1. Run the behavior-focused test prompt         (3-5 min)
2. Read through ALL generated tests              (5-10 min)
3. Flag assertions that test implementation      (during step 2)
4. Rewrite ~30% of assertions                    (5-10 min)
5. Delete tests that only test mocks             (1-2 min)
6. Add missing edge case/error tests manually    (5-10 min)
7. Run the tests, fix any failures               (2-5 min)
Total: ~30 min per module
```

---

## Common Pitfalls

1. **Don't accept the tests without reading every assertion.** AI tests that pass are not the same as AI tests that catch bugs. A test that asserts `toBeDefined()` on everything passes, covers the code, increases your coverage number — and catches zero regressions.

2. **Don't let AI mock internal modules.** If your service calls a utility function in the same package, let it run. Only mock boundaries (database, HTTP, file system, external services).

3. **Don't skip error-path tests.** If the AI generated 15 tests and only 2 test error conditions, add more. Production bugs live in error paths, timeout handling, and concurrent access — not in the happy path.

4. **Don't use AI-generated test data that's obviously fake.** `"test@test.com"` and `"password123"` in test data makes tests harder to read. Use realistic data: `"jane.doe@company.com"`, `"$49.99 monthly subscription"`.

5. **Don't generate tests for trivial code.** Pure utility functions with no dependencies (formatDate, slugify, capitalize) are faster to test manually than to review AI output. Save AI test generation for modules with complex logic, multiple dependencies, and many error paths.
