---
name: audit
description: Scan the full codebase for security vulnerabilities, performance issues, and best practice violations. Produces a prioritized report with severity levels — the developer decides what to fix.
---

Code reviews catch feature-level issues. Audits catch systemic ones — the vulnerabilities, performance problems, and pattern violations that live across the codebase and compound over time.

Run this before major releases, after large development cycles, or periodically as a health check.

## What This Skill Does Not Do

It does not fix anything. It scans, identifies, and reports — letting the developer decide what matters and what to act on. It does not overlap with `/review`, which verifies a specific feature against its plan.

## How to Invoke

```
/audit
```

To scope the audit to specific areas:

```
/audit security      # Security only
/audit performance   # Performance only
/audit practices     # Best practices only
/audit path/to/dir   # Specific directory
```

## How to Scan Efficiently

The checklists below cover 40+ categories. Do not read every file in the codebase — use targeted strategies:

- **Secrets and credentials** — use regex search for patterns (`sk-`, `-----BEGIN`, `token`, `secret`, `password =`, `api_key`). Check `.env` files and lock files, not every source file.
- **Dependencies** — read lock files (`package-lock.json`, `requirements.txt`, `Cargo.lock`, `Gemfile.lock`). Do not scan source for dependency issues.
- **Injection and XSS** — grep for known dangerous patterns (`innerHTML`, `dangerouslySetInnerHTML`, raw SQL interpolation, `exec(`). Do not read every route handler.
- **Auth and authorization** — check route definitions and middleware files. Pattern-match for missing guards on protected routes.
- **Performance (N+1, queries)** — grep for loops containing DB calls, check ORM usage patterns in data-fetching code.
- **Bundle and build** — read config files (webpack, vite, next.config, etc.) and a sample page/component per directory.
- **Configuration and headers** — check a single server/HTTP config file rather than scanning source.
- **For each category**, validate suspected findings before reporting. A matched pattern is not always a real issue.

If you cannot confidently check a category without reading an unreasonable amount of code, flag it as "needs manual review" rather than skipping it silently.

### False Positive Discipline

Audits inevitably produce findings that are not real issues. Follow these rules:

- **Be conservative.** If you are not sure something is a real issue, label it "Unconfirmed — investigation needed" rather than asserting it as a finding.
- **Validate before reporting.** A regex match for "password" in a test fixture is not a secret leak. Read the surrounding context before flagging.
- **When uncertain, lean toward flagging but label the uncertainty.** "Possible hardcoded key — verify this is not a test value." The developer can triage faster than the agent can investigate every match.
- **Distinguish between categories:** a testing-only credential in test code is Minor. A production credential is Critical. Apply the severity guide honestly.

## Phase 1 — Security

Scan for security issues. For each finding, report the file, line, and severity.

### What to check

**Secrets and credentials:**
- Hardcoded API keys, tokens, passwords, connection strings
- `.env` files committed to version control
- Secrets in configuration files, test fixtures, or documentation
- Any string that looks like a credential pattern (sk-, pk-, secret, token, key, password)

**Authentication:**
- Missing or weak auth on protected routes/endpoints
- Hardcoded or predictable session tokens
- Insecure password storage patterns
- Missing rate limiting on auth endpoints
- Weak password requirements or missing password validation

**Authorization:**
- Missing access control checks on API routes
- IDOR (Insecure Direct Object Reference) — users accessing resources they should not
- Role/permission checks missing from protected operations
- Admin endpoints accessible without admin privileges

**Injection:**
- SQL injection — raw SQL queries with string interpolation
- NoSQL injection — unsanitized query parameters
- Command injection — shell commands built from user input
- Template injection — user input in template rendering

**Cross-Site Scripting (XSS):**
- User input rendered without sanitization
- `dangerouslySetInnerHTML` or similar raw HTML insertion
- Unsafe `innerHTML` usage in client-side code
- Missing Content Security Policy headers

**Cross-Site Request Forgery (CSRF):**
- State-changing endpoints without CSRF tokens
- Cookie-based auth without SameSite attributes
- Missing origin/referer validation on sensitive endpoints

**Dependencies:**
- Known vulnerable dependencies (check package.json, requirements.txt, etc.)
- Outdated packages with security patches available
- Unnecessary or unused dependencies that expand the attack surface

**Server-Side Request Forgery (SSRF):**
- User-supplied URLs fetched server-side without validation
- Internal network endpoints accessible via user input
- Missing allowlists for outbound requests

**Data exposure:**
- Sensitive data in API responses (passwords, tokens, PII)
- Over-fetching in GraphQL/REST endpoints
- Missing input validation on user-supplied data
- Insecure data storage or transmission

**Headers and configuration:**
- Missing security headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- CORS configured too permissively
- Debug or info endpoints exposed in production
- Verbose error messages leaking implementation details

### Report format

Every phase uses the same format. Report findings with file, line, and severity:

```
## [Phase Name] Findings

### Critical
- [file:line] — [issue description with evidence]

### Important
- [file:line] — [issue description with evidence]

### Minor
- [file:line] — [issue description with evidence]
```

---

## Phase 2 — Performance

Scan the codebase for performance issues. Report with severity and estimated impact.

### What to check

**Database and queries:**
- N+1 query patterns — loops making individual database queries
- Missing indexes on frequently queried columns
- Inefficient JOINs or subqueries
- Fetching more data than needed (SELECT *)
- No pagination on list endpoints

**Bundle and build:**
- Large dependencies that could be tree-shaken
- Unused imports or dependencies
- Missing code splitting for routes or components
- Large assets (images, fonts) not optimized
- Missing lazy loading for below-fold content

**Rendering and runtime:**
- Expensive computations running on every render
- Unnecessary re-renders in component trees
- Missing memoization for expensive operations
- Large lists rendered without virtualization
- Expensive operations inside loops
- Synchronous blocking operations in async contexts

**Network and caching:**
- Missing caching headers on API responses
- Repeated identical API calls
- Missing request deduplication
- Large payload sizes that could be paginated or filtered
- Missing CDN or cache layer for static assets

**Memory:**
- Event listeners, intervals, or subscriptions not cleaned up
- Growing data structures without bounds
- Circular references preventing garbage collection
- Large objects held in memory longer than needed

### Report format

Use the same format from Phase 1 — section heading, then Critical / Important / Minor with severity and impact.

---

## Phase 3 — Best Practices

Scan the codebase for violations of common engineering best practices.

### What to check

**TypeScript / type safety:**
- Widespread `any` types that could be properly typed
- Missing return types on public functions
- Unsafe type assertions (`as`) without validation
- `@ts-ignore` or `@ts-expect-error` comments

**Error handling:**
- Bare `catch` blocks that swallow errors
- Unhandled promise rejections
- Missing input validation on public APIs
- Errors returned as HTTP 200 instead of appropriate status codes
- Missing fallback or default values for optional dependencies

**Code quality:**
- Dead code — functions, variables, imports that are never used
- Deeply nested conditionals that could be flattened
- Functions doing too many things (violating single responsibility)
- Duplicated logic that could be extracted
- Magic numbers or strings that should be named constants
- Comments explaining "what" instead of "why" (code should be self-documenting)

**Testing:**
- Missing tests for critical paths
- Tests that don't assert anything (no assertions, or always-pass assertions)
- Tests that depend on external services without mocking
- Low test coverage in core business logic
- Integration tests missing for key user flows

**Architecture:**
- Business logic mixed with UI components
- Direct database access from presentation layer
- Circular dependencies between modules
- God objects or god functions with too many responsibilities
- Missing abstraction boundaries between layers

**Consistency:**
- Mixed naming conventions (camelCase, snake_case, kebab-case in same project)
- Inconsistent error response formats
- Inconsistent import/export patterns
- Mixed use of sync and async APIs for similar operations

### Report format

Use the same format from Phase 1.

---

## Full Report

After completing all phases (or the requested subset), produce a summary:

```
## Audit Summary — [project name]

### Security: [X] issues ([C] critical, [I] important, [M] minor)
### Performance: [X] issues ([C] critical, [I] important, [M] minor)
### Best Practices: [X] issues ([C] critical, [I] important, [M] minor)

**Total: [X] issues across [Y] categories**

### Top priorities
1. [Critical issue] — [file:line] — [one-line reason to fix now]
2. [Critical issue] — [file:line] — [one-line reason to fix now]
3. [Important issue] — [file:line] — [one-line reason to fix soon]

The developer owns all fix decisions. Review the full report above
and decide what to act on.
```

## Severity Guide

**Critical — fix before deployment**
- Active security vulnerability (secrets exposed, auth bypass, injection risk)
- Performance issue that degrades user experience measurably
- Architectural violation that will block future development

**Important — fix soon**
- Security hardening (missing headers, permissive CORS)
- Performance optimization with meaningful impact
- Best practice violations that compound over time

**Minor — fix when convenient**
- Informational findings with low risk
- Style and consistency preferences
- Nice-to-have optimizations