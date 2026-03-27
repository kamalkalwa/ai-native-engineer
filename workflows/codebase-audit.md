# Workflow: Codebase Audit with AI

**Time:** ~15 minutes for a mid-sized service (10k-50k lines)
**Tools:** Claude Code (CLI)
**Level:** 2 (AI-Augmented)

---

## When to Use This

- You've inherited a codebase and need to understand its health before making changes
- You're evaluating whether to refactor or rewrite
- You need to build a migration plan and need hard data, not vibes
- A new team member needs to ramp up on technical debt quickly

---

## Step 1: Run the Audit Prompt

Open Claude Code in the root of the repository and run this prompt:

```
Audit this codebase. For each category below, identify specific issues with file:line references.

Categories:
1. **Dead code** — Functions, exports, routes, or components that are defined but never referenced
2. **Security risks** — Hardcoded secrets, SQL injection vectors, missing input validation, insecure dependencies
3. **Error handling gaps** — Functions that can throw but aren't wrapped, missing error boundaries, swallowed errors
4. **Performance concerns** — N+1 queries, missing indexes (inferred from query patterns), synchronous operations that should be async, large bundle imports
5. **Type safety issues** — `any` types, missing null checks, type assertions that bypass safety
6. **Test coverage gaps** — Critical business logic paths that have no test file or no meaningful assertions
7. **Dependency health** — Outdated major versions, deprecated packages, packages with known CVEs
8. **Code duplication** — Near-identical logic in multiple places that should be extracted

Output format: For each category, produce a markdown table with these columns:
| Severity | File:Line | Issue | Suggested Fix |

Severity levels:
- CRITICAL: Will cause bugs or security issues in production
- HIGH: Significant technical debt or reliability risk
- MEDIUM: Code quality issue that increases maintenance cost
- LOW: Style or convention inconsistency

After all tables, provide a summary with:
- Total issues by severity
- Top 3 files that need immediate attention
- Estimated effort to address CRITICAL and HIGH issues (in engineer-hours)
```

---

## Step 2: Review the Audit Output

Claude Code will read through the repository and produce tables for each category. What you're looking for:

**Validate the CRITICALs first.** Open each file:line reference and confirm the issue exists. AI occasionally flags intentional patterns as issues (e.g., a catch block that logs and re-throws might be flagged as "swallowed error"). Expect some false positives — always verify before acting on findings.

**Cross-reference dependency issues.** Run these to confirm what Claude Code found:

```bash
# For Node.js projects
npx npm-audit-resolver
npm outdated

# For Python projects
pip-audit
pip list --outdated
```

**Check dead code findings against tests.** Sometimes Claude Code flags functions as dead code that are only referenced in test files. This is still useful information (it means the function is tested but unused in production) but the severity is LOW, not MEDIUM.

---

## Step 3: Feed Audit Results into a Migration Plan

After you've reviewed and confirmed the audit findings, use this follow-up prompt:

```
Based on the audit results above, create a prioritized migration plan.

Rules:
1. Group work into phases. Each phase should be independently deployable.
2. Phase 1 must contain ONLY critical security fixes and data-integrity issues.
3. Phase 2 should address HIGH-severity issues that reduce reliability risk.
4. Phase 3+ handles MEDIUM and LOW issues grouped by area (all error handling together, all type safety together, etc.)
5. For each phase, specify:
   - Files to change
   - Type of change (fix, refactor, delete, add tests)
   - Dependencies between changes (what must be done before what)
   - Estimated time (in hours, not days — be specific)
   - Risk level of the change (what could break)
   - Rollback strategy if something goes wrong

Output as a numbered task list per phase, not prose.
```

---

## Expected Output Format

Here's what the audit output looks like for a real-world codebase. This example is from a hypothetical Express + React app (a SaaS billing dashboard).

### Category 1: Security Risks

| Severity | File:Line | Issue | Suggested Fix |
|----------|-----------|-------|---------------|
| CRITICAL | `server/config/db.ts:14` | Database password hardcoded as `const DB_PASS = "prodpass123"` | Move to environment variable, add to `.env.example` without value |
| CRITICAL | `server/routes/admin.ts:45` | Admin endpoint `/api/admin/users` has no authentication middleware | Add `requireRole('admin')` middleware before route handler |
| CRITICAL | `server/routes/billing.ts:78` | Stripe webhook endpoint does not verify webhook signature | Add `stripe.webhooks.constructEvent()` with signing secret |
| HIGH | `server/middleware/auth.ts:23` | JWT token never expires — `expiresIn` option is not set | Set `expiresIn: '24h'` in `jwt.sign()` options |
| HIGH | `server/routes/users.ts:112` | User input passed directly to SQL query via string interpolation | Use parameterized query: `db.query('SELECT * FROM users WHERE id = $1', [userId])` |
| MEDIUM | `client/src/utils/api.ts:8` | API base URL hardcoded to `http://localhost:3001` | Read from `REACT_APP_API_URL` environment variable |

### Category 2: Error Handling Gaps

| Severity | File:Line | Issue | Suggested Fix |
|----------|-----------|-------|---------------|
| HIGH | `server/services/billingService.ts:34-67` | `createSubscription()` calls Stripe API without try/catch. Stripe errors will crash the process. | Wrap in try/catch, return `Result<Subscription, BillingError>` |
| HIGH | `server/services/emailService.ts:19` | `sendWelcomeEmail()` is `async` but caller at `server/routes/auth.ts:89` does not await it or catch rejections | Either await with error handling, or add `.catch(logError)` for fire-and-forget |
| MEDIUM | `client/src/pages/Dashboard.tsx:45` | No error boundary around the billing widget. API failure renders blank page. | Wrap in `<ErrorBoundary>` with fallback UI |
| MEDIUM | `server/routes/billing.ts:23` | `catch (e) { res.status(500).send('Error') }` — swallows error details, no logging | Log the error with context, return structured error response |

### Category 3: Dead Code

| Severity | File:Line | Issue | Suggested Fix |
|----------|-----------|-------|---------------|
| LOW | `server/services/legacyPayment.ts:1-145` | Entire file is unreferenced after migration to Stripe in PR #234 | Delete file, remove from `tsconfig.json` includes |
| LOW | `client/src/components/OldPricingTable.tsx:1-89` | Component not imported anywhere. `NewPricingTable.tsx` replaced it. | Delete file |
| LOW | `server/utils/dateHelpers.ts:45-67` | `formatDateLegacy()` function not called anywhere | Delete function |

### Category 4: Performance Concerns

| Severity | File:Line | Issue | Suggested Fix |
|----------|-----------|-------|---------------|
| HIGH | `server/routes/dashboard.ts:34` | Fetches all invoices for a user, then filters in JS. No pagination, no WHERE clause on date range. | Add date range filter to SQL query, add `LIMIT/OFFSET` pagination |
| MEDIUM | `server/routes/users.ts:56-70` | Loop fetches user profile, then subscriptions, then invoices in 3 separate queries per user (N+1) | Use a JOIN query or batch fetch with `WHERE user_id IN (...)` |
| MEDIUM | `client/src/App.tsx:3` | `import moment from 'moment'` — entire 67KB moment library loaded for one `format()` call | Replace with `date-fns/format` (2KB) |

### Category 5: Type Safety Issues

| Severity | File:Line | Issue | Suggested Fix |
|----------|-----------|-------|---------------|
| HIGH | `server/services/billingService.ts:12` | `function processWebhook(event: any)` — Stripe event has well-defined types | Use `Stripe.Event` from the `stripe` package types |
| MEDIUM | `server/middleware/auth.ts:34` | `(req as any).user = decoded` — extends Request without type declaration | Create `types/express.d.ts` with `declare module 'express'` augmentation |
| MEDIUM | `client/src/hooks/useApi.ts:23` | Return type is `Promise<any>` — consumers have no type safety | Add generic type parameter: `useApi<T>(url: string): Promise<T>` |

### Summary

| Severity | Count |
|----------|-------|
| CRITICAL | 3 |
| HIGH | 7 |
| MEDIUM | 7 |
| LOW | 3 |

**Top 3 files needing immediate attention:**
1. `server/routes/billing.ts` — unverified webhook, swallowed errors
2. `server/config/db.ts` — hardcoded production credentials
3. `server/services/billingService.ts` — untyped Stripe events, no error handling on API calls

**Estimated effort for CRITICAL + HIGH fixes:** 6-8 engineer-hours

---

## Common Pitfalls

1. **Don't run this on a monorepo root.** Scope it to a single service or package. Running on a 200k-line monorepo will hit context limits and produce shallow results. Run it per-package instead.

2. **Don't skip the manual verification step.** The audit is a starting point, not a final report. Some findings will be false positives — patterns that look wrong but are intentional.

3. **Don't try to fix everything at once.** Use the migration plan output to phase the work. Fixing CRITICAL issues in Phase 1 is non-negotiable. Everything else can be scheduled into sprints.

4. **Re-run after major changes.** After completing Phase 1 fixes, run the audit again. New issues sometimes emerge when you fix one layer and expose another.
