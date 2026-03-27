---
name: test-gen
description: Generate behavior-based tests with built-in guardrails against common AI test anti-patterns. Use when adding tests to a module or generating tests for new code.
argument-hint: [file-or-module-path]
allowed-tools: Read, Grep, Glob, Write, Bash(npx *), Bash(npm *)
---

# Test Generation

Generate behavior-based tests for the file or module the user specifies. Follow these rules strictly — they prevent the most common AI test anti-patterns.

## Rules

1. **Test BEHAVIOR, not implementation.** Each test name completes "it should ___" with something a product manager would understand.
   - GOOD: "it should return an error when the email is already registered"
   - BAD: "it should call userRepository.findByEmail with the correct argument"

2. **Test the PUBLIC interface only.** Do not test private methods, internal state, or how the function achieves its result.

3. **For each function, test these categories:**
   - Happy path (valid input → expected output)
   - Invalid input (each type of bad input → appropriate error/response)
   - Edge cases (empty arrays, null values, boundary numbers, unicode strings)
   - Error conditions (database down, API timeout, network failure)

4. **Mock EXTERNAL dependencies only** (database, APIs, file system). If function A calls function B in the same service, do NOT mock function B.

5. **Use descriptive test data.**
   - GOOD: `const email = "already-registered@company.com"`
   - BAD: `const email = "test@test.com"`

6. **Assert on COMPLETE return values** when practical.
   - GOOD: `expect(result).toEqual({ id: "user-1", email: "new@co.com", role: "member" })`
   - BAD: `expect(result.id).toBeDefined()`

7. **For error cases, test useful information**, not just that it threw.
   - GOOD: `expect(error.message).toContain("User with email already exists")`
   - BAD: `expect(fn).toThrow()`

8. **Each test must be independent.** No shared mutable state. Setup goes in beforeEach.

## After Generation

After generating the tests, tell the user:

"Review every assertion. Expect to rewrite ~30% of them. Watch for:
- Assertions that test mock call counts (implementation detail)
- `toBeDefined()` or `toBeTruthy()` (too weak — assert specific values)
- Snapshot assertions on dynamic data (assert specific fields instead)
- Tests that pass even if the function is deleted (testing the mock, not the code)

Want me to walk through each assertion and flag the ones that likely need rewriting?"

## Anti-Patterns to Avoid

Never generate tests that:
- Use `toHaveBeenCalledTimes()` unless testing rate limiting or retry logic
- Access private/internal state via `(service as any)._cache`
- Use `toMatchSnapshot()` on objects with dynamic fields
- Have more than 80% happy-path tests (error paths are where production bugs live)
