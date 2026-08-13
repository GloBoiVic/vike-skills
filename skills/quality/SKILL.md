---
name: quality
description: Diagnose failures and choose a safe recovery path; reproduce before fixing and escalate feature verification or systemic risk to the canonical skills.
---

# Quality

Use only for debugging a failure or recovering from a bad implementation session. Use `review` for feature verification and `audit` for codebase-wide security, performance, or practice audits.

## Choose a path

- **Targeted debugging:** an isolated failure on its first or second fix attempt.
- **Hard reset:** repeated failed fixes have polluted the session; preserve verified work, write a reset note, stop, and begin clean.
- **Rethink:** the requirement, library, or architecture assumption is wrong; stop implementation and obtain an agreed new blueprint from `architect`.

## Targeted debugging

1. Reproduce first. If not reproducible, request expected/actual behavior, steps, and errors.
2. Isolate the file, line, function, or component and record the evidence.
3. Trace inputs → transformations → outputs to state the causal root cause, not the symptom.
4. Describe the minimal fix, change only the agreed scope, then rerun reproduction and relevant tests. Do not stack patches or fix unrelated issues.
5. For a non-trivial bug, add a regression test or scan for similar patterns and record a prevention action.

Report:

```text
Isolated to: [file:line] — [symbol]
Evidence: [proof]
Root cause: [cause]
How found: [analysis path]
Prevention note: [why it escaped]
Action: [test, validation, documentation, or guard]
```

## Recovery notes

For a polluted session record what was built, what failed, what to avoid, and the clean starting point. For a wrong foundation record assumed/reality/correct approach and what to discard/keep. Do not reset or delete dispatch state. After a successful behavior or architecture change, run `review`; for auth, payments, security, or systemic risk, escalate to premium review and `audit`.
