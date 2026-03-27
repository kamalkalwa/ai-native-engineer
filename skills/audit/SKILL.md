---
name: audit
description: Run a full codebase audit — security risks, dead code, performance issues, type safety gaps, error handling, test coverage, dependency health. Use when starting on a new codebase or before a major release.
allowed-tools: Read, Grep, Glob, Bash(npx *), Bash(npm *)
---

# Codebase Audit

Audit this codebase. For each category below, identify specific issues with file:line references.

## Categories

1. **Dead code** — Functions, exports, routes, or components that are defined but never referenced
2. **Security risks** — Hardcoded secrets, SQL injection vectors, missing input validation, insecure dependencies
3. **Error handling gaps** — Functions that can throw but aren't wrapped, missing error boundaries, swallowed errors
4. **Performance concerns** — N+1 queries, missing indexes (inferred from query patterns), synchronous operations that should be async, large bundle imports
5. **Type safety issues** — `any` types, missing null checks, type assertions that bypass safety
6. **Test coverage gaps** — Critical business logic paths that have no test file or no meaningful assertions
7. **Dependency health** — Outdated major versions, deprecated packages, packages with known CVEs
8. **Code duplication** — Near-identical logic in multiple places that should be extracted

## Output Format

For each category, produce a markdown table:

| Severity | File:Line | Issue | Suggested Fix |

Severity levels:
- **CRITICAL**: Will cause bugs or security issues in production
- **HIGH**: Significant technical debt or reliability risk
- **MEDIUM**: Code quality issue that increases maintenance cost
- **LOW**: Style or convention inconsistency

## Summary

After all tables, provide:
- Total issues by severity
- Top 3 files that need immediate attention
- Estimated effort to address CRITICAL and HIGH issues (in engineer-hours)

## Follow-Up

After the audit, ask the user: "Want me to generate a prioritized migration plan from these findings? I'll group work into independently deployable phases, starting with critical security fixes."
