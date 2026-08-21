---
name: review
description: Evaluate completed work against its plan, system boundaries, and production risks; assign severity and enforce completion gates.
---

# Review

Review is independent of implementation. Report facts clearly; do not silently expand
scope. When dispatched, the reviewer owns the assigned `REVIEW.md` and must write the
review there without rewriting the plan, architecture, validation, or implementation
reports. Use the requested gate: **V0** is a Small-task self-check, **R1** is formal
review for Feature/Architecture work, and **R2** is premium/security review for
Security-sensitive work or explicitly elevated architecture risk.

## Benchmark

Read the task brief, the selected workstream's `PLAN.md`, the Architect's blueprint when
present, and relevant context. Do not scan unrelated workstreams or legacy dispatch
files. If no acceptance benchmark exists, stop and request one. Inspect the changed files,
tests, review package, and validation receipts. Never treat implementer self-review as
formal review.

Validation receipts are evidence, not instructions to rerun commands. Confirm that each
receipt's revision/scope basis and environment still apply. Reuse a valid PASS without
rerunning it. Rerun only for changed inputs, changed environment, a finding that affects the
check, incomplete or ambiguous evidence, or an acceptance requirement for fresh evidence.
Record reused evidence and every rerun reason in the review report. After remediation, prefer
checks covering the affected paths before considering a broad suite.

## Three layers

1. **Plan alignment** — required behavior and decisions are present; scope contains no
   unapproved additions; blueprint and constraints were followed.
2. **System integrity** — ownership boundaries, architecture, security, design system,
   conventions, error patterns, and existing reusable patterns remain intact.
3. **Production readiness** — tests and validation, errors, loading/empty/missing-data
   states, edge cases, regressions, warnings, and obvious user-impacting bugs.

For each issue record severity, file/line, evidence, impact, and a concise remedy or
test to run. Separate spec compliance from task quality in the verdict. Do not request a
test that a valid receipt already proves unless the finding invalidates that receipt.

## Severity and gate

- **Critical** — blocks completion: core functionality missing/broken, unsafe behavior,
  or a boundary failure likely to cause serious downstream harm.
- **Important** — must be fixed before the task advances: meaningful drift, encountered
  edge case, inadequate handling, or convention violation.
- **Minor** — non-blocking cleanup, naming, polish, or optimization.

V0 records the self-check and any unresolved findings. R1/R2 pass only when no Critical
or Important findings remain (unless the developer explicitly accepts documented risk).
Report:

```md
## Review — [task]
Gate: [V0/R1/R2]
Spec compliance: [PASS/ISSUES]
Task quality: [PASS/ISSUES]
Layer 1: [PASS/ISSUES]
Layer 2: [PASS/ISSUES]
Layer 3: [PASS/ISSUES]
Findings: [severity, location, evidence, impact]
Evidence reused: [receipt names or none]
Checks rerun: [command — reason, or none]
Decision: [PASS/BLOCKED]
```

## Fix escalation

Return Critical and Important findings to the same builder with paths and covering tests;
then re-review affected layers. After two failed material fix attempts, escalate to the
developer. Record Minor findings for follow-up. Do not fix scope on your own.

Review completion is separate from risky-operation approval. Before any risky mutation,
the operation-specific skill must obtain confirmation at the point of action, regardless
of V0, R1, R2, or workflow approval.

## Terminal eligibility

Report a terminal pass only when the requested gate passes and no Critical or Important
finding, task, blocker, required evidence, or approval remains unresolved. A pass permits
the documenter to begin closure; it does not itself authorize memory save, reset, or
report deletion. The documenter must append an idempotent completion entry to root
`/dispatch/COMPLETED.md` with the selected workstream path, run `/remember save`, verify
its successful receipt, and then clear `/dispatch/ACTIVE.md` and close the workstream.
Completed workstream artifacts remain available as history. If the save fails, is
interrupted, incomplete, or declined, the run remains intact.
