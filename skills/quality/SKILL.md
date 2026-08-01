---
name: quality
description: Code quality operations — systematic debugging, failure recovery, code review, security audit, and performance scanning. Diagnose before acting.
---

This skill replaces the need for separate debug, recover, review, and audit skills. It covers five sections. Use the one that matches the current situation.

## Section Selection

- **Bug to fix?** → Section A (Debugging)
- **Session or approach gone wrong?** → Section B (Recovery)
- **Feature just built, needs review?** → Section C (Review)
- **Whole codebase security check?** → Section D (Security Audit)
- **Performance bottleneck?** → Section E (Performance Audit)

---

## Section A — Systematic Debugging

Not every bug is obvious. Replace guessing with a systematic process. Four phases, followed in order.

### Phase 1 — Reproduce and Isolate

**Step 1.1 — Reproduce it yourself first**
Run the relevant code, check the error, confirm you can see the wrong behaviour.

**Step 1.2 — Ask only if you cannot reproduce**
What was expected? What happened instead? Exact steps? Error messages?

**Step 1.3 — Isolate to a specific location**
Narrow to a specific file, function, or component.
```
Isolated to: [file:line] — [function/component name]
Evidence: [what confirms this is the location]
```
Do not proceed until the failure is isolated.

### Phase 2 — Root Cause Analysis

**Step 2.1 — Read the code at the isolated location**
**Step 2.2 — Trace the data flow** — inputs, transformations, outputs, divergence point
**Step 2.3 — Identify the root cause**
```
Root cause: [specific, actionable statement]
How it was found: [brief analysis path]
Why this is the root cause and not a symptom: [cause-effect chain]
```

### Phase 3 — Fix and Verify

**Step 3.1 — Design the fix first**, then present to developer
**Step 3.2 — Apply the minimal change**
**Step 3.3 — Verify** — reproduction steps pass, existing tests pass
**Step 3.4 — Confirm with developer**

### Phase 4 — Defense in Depth

**Step 4.1 — Add a regression test** if none exists
**Step 4.2 — Scan for similar patterns** elsewhere in the codebase
**Step 4.3 — Reflect on prevention** — what allowed this bug?

```
Prevention note: [what would prevent this class of bug]
Action: [add validation, clarify docs, add tests, etc.]
```

**When to stop:**
- Phase 1 — cannot reproduce → ask developer
- Phase 2 — cannot find root cause → escalate
- Phase 3 — fix fails → return to Phase 2
- Phase 4 — scale effort to bug complexity

---

## Section B — Failure Recovery

Not every problem is a bug. Diagnose the failure mode before acting.

### Step 1 — Describe what went wrong
Ask: what was expected, what happened, how many fix attempts?

### Step 2 — Identify the failure mode

**Failure Mode 1 — Specific thing is broken**
- Isolated problem, rest works, first/second fix attempt
- Response: Targeted fix → go to Section A (Debugging)

**Failure Mode 2 — Session has gone wrong**
- Multiple fix attempts made things worse, code tangled, context polluted
- Response: Hard reset → save reset note, end session, start fresh

**Failure Mode 3 — Foundation is wrong**
- Code runs but approach misunderstands requirement, library, or architecture
- Response: Rethink → run architect skill before any new code

### Step 3 — Execute the response

**For Mode 2 — Hard Reset:**
Save a reset note:
```
## Reset Note — [Feature Name]
### What we were building
### What went wrong
### What to avoid next time
### Starting point for next session
```
Instruct developer to end session and start fresh.

**For Mode 3 — Rethink:**
```
The core issue is not a bug — it is a wrong assumption:
Assumed: [what was assumed]
Reality: [what is actually true]

Correct approach: [description]
Key difference: [explanation]
```
Do not rebuild until developer confirms.

---

## Section C — Feature Review

Review in three layers after a feature is built.

### Layer 1 — Does it match the plan?
- Every requirement present?
- Decisions reflected in code?
- Scope maintained (nothing extra, nothing missing)?

### Layer 2 — Does it respect the system?
- Architecture boundaries respected?
- Design system tokens used (no hardcoded values)?
- Code standards followed?
- Existing patterns used (not new patterns)?

### Layer 3 — Is it production ready?
- Error handling — caught and graceful?
- Edge cases — empty states, loading, missing data?
- Console errors?
- Obvious bugs?

### Report format
```
## Review — [Feature Name]
### Layer 1 — Plan alignment: [PASS / ISSUES FOUND]
### Layer 2 — System integrity: [PASS / ISSUES FOUND]
### Layer 3 — Production readiness: [PASS / ISSUES FOUND]
### Summary
[X] issues across [Y] layers.
```
Write to /dispatch/REVIEW.md.

**Severity:**
- Critical — fix before moving on
- Important — fix soon
- Minor — fix when convenient

Critical issues: offer to fix immediately. Others: report and let developer decide.

---

## Section D — Security Audit

Scan the codebase for security vulnerabilities. Write findings to /dispatch/REVIEW.md.

**Check:**
- Secrets and credentials hardcoded
- Missing authentication on protected routes
- Missing authorization checks (IDOR)
- SQL/NoSQL injection, command injection, XSS
- Missing CSRF protection
- Known vulnerable dependencies
- SSRF — user-supplied URLs fetched server-side
- Sensitive data in API responses
- Missing security headers (CSP, HSTS, X-Frame-Options)
- Permissive CORS

**Report format:**
```
## Security Findings
### Critical
- [file:line] — [issue with evidence]
### Important
- [file:line] — [issue with evidence]
### Minor
- [file:line] — [issue with evidence]
```

**Severity guide:**
- Critical — active vulnerability, fix before deployment
- Important — hardening, fix soon
- Minor — informational, fix when convenient

---

## Section E — Performance Audit

Scan for performance issues. Write findings to /dispatch/REVIEW.md.

**Check:**
- N+1 query patterns
- Missing indexes
- SELECT * fetching more than needed
- No pagination on list endpoints
- Large dependencies not tree-shaken
- Missing code splitting
- Expensive operations inside loops or every render
- Missing memoization
- Large lists without virtualization
- Missing caching headers
- Repeated identical API calls
- Memory leaks (event listeners not cleaned up)

**Report format:**
```
## Performance Findings
### Critical
- [file:line] — [issue with evidence and impact estimate]
### Important
- [file:line] — [issue with evidence and impact estimate]
### Minor
- [file:line] — [issue with evidence and impact estimate]
```

---

## Section Selection Quick Reference

| Symptom | Section |
|---------|---------|
| Bug to fix | A — Debugging |
| Session went wrong / multiple failed attempts | B — Recovery |
| Wrong approach / foundation | B — Recovery → Rethink |
| Feature needs verification | C — Review |
| Security vulnerability check | D — Security Audit |
| Performance bottleneck | E — Performance Audit |
