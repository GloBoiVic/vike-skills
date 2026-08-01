---
name: quality
description: Systematic debugging and failure recovery. Diagnose the cause before fixing; hand feature reviews to review and systemic audits to audit.
---

# Quality — Debugging and Recovery

Use this skill only for debugging a failure or deciding how to recover from a bad implementation session. It does not replace the canonical `review` or `audit` skills.

## Choose the response

- Isolated bug, first or second attempt → **Targeted debugging**
- Multiple failed fixes have polluted the session → **Hard reset**
- The implementation is based on a wrong requirement, library, or architecture → **Rethink**

For feature verification, use `review`. For security, performance, or codebase-wide audits, use `audit`.

## Targeted Debugging

### 1. Reproduce and isolate

Reproduce the failure before changing code. If it cannot be reproduced, ask for exact expected behavior, actual behavior, steps, and errors. Isolate the failure to a specific file, line, function, or component.

```text
Isolated to: [file:line] — [function/component]
Evidence: [what confirms the location]
```

### 2. Find the root cause

Trace inputs, transformations, outputs, and the point where actual behavior diverges. State the cause, not the symptom:

```text
Root cause: [specific actionable explanation]
How found: [analysis path]
Why this is causal: [cause-effect chain]
```

### 3. Fix and verify

Describe the minimal fix before applying it. Then run the reproduction steps and relevant tests. Do not fix unrelated issues during the same task.

### 4. Prevent recurrence

For non-trivial bugs, add a regression test, scan for similar patterns, and record:

```text
Prevention note: [what allowed the bug]
Action: [specific prevention: validation, documentation, test, or guard]
```

If the fix fails, return to root-cause analysis rather than stacking patches.

## Recovery

### Failure Mode 1 — Specific thing is broken

Use targeted debugging above.

### Failure Mode 2 — Session is polluted

Signs: repeated failed fixes, tangled code, unclear original problem. Save a reset note, preserve only verified useful work, end the session, and start a clean session. Do not keep patching.

```markdown
## Reset Note — [Feature]
### What we were building
### What went wrong
### What to avoid next time
### Starting point for the fresh session
```

### Failure Mode 3 — Foundation is wrong

Name the wrong assumption, explain reality, and stop implementation:

```text
Assumed: [wrong assumption]
Reality: [correct understanding]
Correct approach: [new direction]
What to discard: [invalid work]
What to keep: [verified work]
```

Run `architect` and wait for agreement before rebuilding.

## Handoffs

- After a successful fix: run `review` if behavior or architecture changed.
- For authentication, payments, security, or systemic risk: escalate to `reviewer-premium` and `audit`.
- Record discoveries and prevention decisions in `/dispatch/DECISIONS.md`.
