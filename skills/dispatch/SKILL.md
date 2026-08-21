---
name: dispatch
description: Execute an approved plan with isolated subagents, sequential writers, review gates, ledgers, and bounded fix loops.
---

# Dispatch

Execute an approved plan; do not redesign it. Read the active dispatch control plane and
the current workstream artifacts needed for the next transition; do not bulk-read
historical workstreams or reports. Resume completed tasks, and never re-dispatch them.
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

For each task in plan order: issue a precise brief, assigned artifact path, and worktree cwd; dispatch one fresh
implementer with the authoritative blueprint; collect its status, concerns, report, and validation
receipts; package the diff for review; run V0/R1/R2 as classified; return Critical/Important
findings to the same builder and re-review. After two failed material fixes, escalate.
Record task state in the orchestrator-owned control plane, while each agent writes its
assigned workstream artifact. Writers, tests, reviews, and documentation are strictly
sequential; only bounded, one-level, read-only exploration may fan out to three workers.

## Dispatch bootstrap and ownership

`/init` initializes project context only. For a non-small request with no `/dispatch/`,
the orchestrator creates the directory, `ACTIVE.md`, `PLAN.md`, and `TASKS.md`, plus a
`workstreams/` location only when a separate workstream record is justified. A feature
ID is optional; use a descriptive workstream name and derive a slug only for a path.
Small work does not create dispatch state unless tracking is requested.

The active control plane is `/dispatch/ACTIVE.md`, `/dispatch/PLAN.md`, and
`/dispatch/TASKS.md`. The ownership table is authoritative:

| Artifact | Owner |
|---|---|
| `ACTIVE.md`, `PLAN.md`, `TASKS.md` | Orchestrator |
| `EXPLORATION.md` | Explore |
| `RESEARCH.md` | Research, when assigned |
| `ARCHITECTURE.md` | Architect |
| `READY.md` | Worktrees |
| `TASK-*.md` | Assigned builder |
| `VALIDATION.md` | Tester or check runner |
| `AUDIT.md` | Audit |
| `REVIEW.md` | Reviewer |
| `RECORD.md`, `COMPLETED.md` | Documenter |

An agent may write application files within its approved implementation scope and its
assigned dispatch artifact only. It may read other artifacts but must not rewrite them,
change their conclusions, or silently create a replacement. If an owned artifact is
missing or materially incorrect, stop and return the issue to its owner.

## Validation evidence

Every check that may be reused by a later agent must have a receipt in the task report or
`REVIEW.md`:

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

`MODEL-LOG.md` should record at least `Task`, `Agent`, `Model`, `Outcome`, `Validation`,
`Reused evidence`, and `Rerun reason`. Keep validation details concise and link to the task
report or `REVIEW.md` for the full receipt rather than copying long command output.

Statuses: `DONE` proceeds to review; `DONE_WITH_CONCERNS` resolves concerns first;
`NEEDS_CONTEXT` supplies context and re-dispatches; `BLOCKED` escalates or revises the
plan with the human; `DISCOVERY` records the discovery and updates remaining tasks.

## Terminal lifecycle

Terminal eligibility requires every planned task done, required tests/evidence recorded,
the applicable review gate passed with no unresolved Critical/Important finding, no
blocker or pending approval, and no interruption. The documenter then:

1. Reconciles one idempotent completion record in `COMPLETED.md`, including a precise
   inventory of one-off reports and their material summaries.
2. Runs `/remember save` and verifies a successful save receipt.
3. Records that receipt in `COMPLETED.md` before any reset.
4. Resets only active control files (`ACTIVE.md`, `PLAN.md`, `TASKS.md`, and any
   explicitly designated active scratch files) to their minimal templates.
5. Deletes only explicitly designated active scratch reports whose material content is
   covered by the completion record. Role-owned workstream artifacts, unknown files, and
   user-authored reports remain.

Steps are retry-safe: an existing completion record, receipt, reset, or deletion is
recognized rather than duplicated; a failed, incomplete, interrupted, or declined save
never permits reset or deletion. `COMPLETED.md` is a compact durable completion index;
the workstream `RECORD.md` and role-owned artifacts preserve the supporting history.

Redact secrets everywhere. Do not add unapproved runtime dependencies or caches; the
workstream structure described above is the approved dispatch organization.

## Git boundaries

Dispatch never performs or implies automatic commits, pushes, merges, branch deletion, or
worktree cleanup. A writer may not switch checkout or branch. If the READY receipt no longer
matches the assigned cwd, branch, SHA, or scope, block the task and return to `worktrees`
for recovery and a newly confirmed setup. Workflow approval and dispatch approval never
 authorize repository-changing Git operations.
