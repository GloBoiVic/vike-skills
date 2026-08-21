---
name: dispatch
description: Execute an approved plan with isolated subagents, sequential writers, review gates, ledgers, and bounded fix loops.
---

# Dispatch

Execute an approved plan; do not redesign it. Read `/dispatch/ACTIVE.md` first, then only
the selected workstream directory and the artifacts required for the next transition. Do
not bulk-read historical workstreams, legacy dispatch files, or unrelated reports. Resume
completed tasks, and never re-dispatch them.
For non-small work, verify Explore,
Architect, and recorded human confirmation, then require a valid `READY` receipt from
`worktrees` before dispatching any writer. The receipt must contain root, path, dedicated
feature branch, full SHA, scope, status, context, and recovery; missing, stale, or
mismatched readiness blocks dispatch. Pass the precise worktree `cwd`, context manifest,
and receipt scope to every
implementation, test, review, and documentation agent. Small work runs in the current
 checkout unless isolation was requested. Read-only Git inspection may run without
 separate confirmation; the operation-specific skill still confirms each exact
repository-changing Git command immediately before it occurs. Never assume that
uncommitted files in another checkout are available in the assigned worktree.

## Loop

For each task in plan order: issue a precise brief, selected workstream root, assigned
artifact path, required inputs, forbidden dispatch paths, and worktree cwd; dispatch one
fresh implementer with the authoritative blueprint; collect its status, concerns, report, and validation
receipts; package the diff for review; run V0/R1/R2 as classified; return Critical/Important
findings to the same builder and re-review. After two failed material fixes, escalate.
Record task state in the workstream `PLAN.md`, while each agent writes its assigned
workstream artifact. Writers, tests, reviews, and documentation are strictly
sequential; only bounded, one-level, read-only exploration may fan out to three workers.

## Dispatch bootstrap and ownership

`/init` initializes project context only. For a non-small request with no `/dispatch/`,
the orchestrator creates `/dispatch/ACTIVE.md`, `/dispatch/COMPLETED.md`, and a
descriptive workstream directory under `/dispatch/workstreams/<slug>/`. The workstream
contains `PLAN.md`, `EXPLORATION.md`, `ARCHITECTURE.md`, `READY.md`, `VALIDATION.md`,
and `REVIEW.md`; add `TASK-*.md`, `RESEARCH.md`, or `AUDIT.md` only when needed. A
feature ID is optional; use a descriptive workstream name and derive a slug only for a
path. Small work does not create dispatch state unless tracking is requested.

`ACTIVE.md` is the only root pointer to current work. It must identify the selected
workstream path, phase, owner, required artifact, and next transition. The ownership
table is authoritative:

| Artifact | Owner |
|---|---|
| `ACTIVE.md`, workstream `PLAN.md` | Orchestrator |
| `EXPLORATION.md` | Explore |
| `RESEARCH.md` | Research, when assigned |
| `ARCHITECTURE.md` | Architect |
| `READY.md` | Worktrees |
| `TASK-*.md` | Assigned builder |
| `VALIDATION.md` | Tester or check runner |
| `AUDIT.md` | Audit |
| `REVIEW.md` | Reviewer |
| root `COMPLETED.md` index | Documenter |

An agent may write application files within its approved implementation scope and its
assigned artifact only. Its brief must name the workstream root, required inputs, and
forbidden dispatch paths. It may read other artifacts in the selected workstream but
must not rewrite them, change their conclusions, or silently create a replacement. It
must not scan unrelated workstreams. If an owned artifact is missing or materially
incorrect, stop and return the issue to its owner.

## Validation evidence

Every check that may be reused by a later agent must have a receipt in `VALIDATION.md`,
the assigned task report, or `REVIEW.md`:

```md
### Validation Receipt — [name]
- Command: `[exact command]`
- Result: PASS | FAIL | BLOCKED
- Scope: [tests/checks and relevant paths]
- Revision/scope basis: [full SHA, or explicit changed-file list for uncommitted work]
- Environment: [runtime, database, service, network, or other relevant condition]
- Time: [timestamp]
- Reusable until: [scope change, environment change, or explicit invalidation]
```

Reviewers and testers must inspect applicable receipts before running commands. A PASS may be
reused when its revision/scope basis and environment remain valid. Do not rerun an unchanged
check merely to reproduce a prior PASS. Rerun only when the relevant files changed, the
environment changed, a finding affects the check, the receipt is incomplete/ambiguous, or the
acceptance gate explicitly requires fresh evidence. Record `Reused evidence: [receipt]` or
`Rerun reason: [reason]` in the report. After a fix, prefer targeted validation for affected
paths; run broad suites only when the change or acceptance criteria justify them.

The task or validation artifact should record `Task`, `Agent`, `Model`, `Outcome`,
`Validation`, `Reused evidence`, and `Rerun reason` when applicable. There is no separate
model ledger. Keep validation details concise rather than copying long command output.

Statuses: `DONE` proceeds to review; `DONE_WITH_CONCERNS` resolves concerns first;
`NEEDS_CONTEXT` supplies context and re-dispatches; `BLOCKED` escalates or revises the
plan with the human; `DISCOVERY` records the discovery and updates remaining tasks.

## Terminal lifecycle

Terminal eligibility requires every planned task done, required tests/evidence recorded,
the applicable review gate passed with no unresolved Critical/Important finding, no
blocker or pending approval, and no interruption. The documenter then:

1. Reconciles one idempotent completion entry in root `COMPLETED.md`, including the
   workstream path, outcome, validation summary, and any deferred work.
2. Runs `/remember save` and verifies a successful save receipt.
3. Records that receipt in `COMPLETED.md` before any reset.
4. Clears or resets `ACTIVE.md` so no workstream remains marked active.
5. Marks the selected workstream closed. Preserve its role-owned artifacts as history;
   do not delete them merely because the feature is complete.

Steps are retry-safe: an existing completion record, receipt, reset, or deletion is
recognized rather than duplicated; a failed, incomplete, interrupted, or declined save
never permits reset or deletion. `COMPLETED.md` is a compact durable completion index;
the closed workstream directory preserves supporting history.

Redact secrets everywhere. Do not add unapproved runtime dependencies or caches; the
workstream structure described above is the approved dispatch organization.

## Git boundaries

Dispatch never performs or implies automatic commits, pushes, merges, branch deletion, or
worktree cleanup. A writer may not switch checkout or branch. If the READY receipt no longer
matches the assigned cwd, branch, SHA, or scope, block the task and return to `worktrees`
for recovery and a newly confirmed setup. Workflow approval and dispatch approval never
 authorize repository-changing Git operations.
