---
name: dispatch
description: Execute an approved plan with isolated subagents, sequential writers, review gates, ledgers, and bounded fix loops.
---

# Dispatch

Execute an approved plan; do not redesign it. Read all existing flat `/dispatch/` files,
resume completed tasks, and never re-dispatch them. For non-small work, verify Explore,
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

For each task in plan order: issue a precise brief, report path, and worktree cwd; dispatch one fresh
implementer with the authoritative blueprint; collect its tests, status, concerns, and
report; package the diff for review; run V0/R1/R2 as classified; return Critical/Important
findings to the same builder and re-review. After two failed material fixes, escalate.
Record task, findings, and model usage in the flat files. Writers, tests, reviews, and
documentation are strictly sequential; only bounded, one-level, read-only exploration
may fan out to three workers.

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
4. Resets only `PLAN.md`, `TASKS.md`, `ARCHITECTURE.md`, `REVIEW.md`, `EXPLORATION.md`,
   `DECISIONS.md`, and `MODEL-LOG.md` to their minimal templates.
5. Deletes only inventoried reports whose material content is covered by the completion
   record. Unknown or user-authored reports remain.

Steps are retry-safe: an existing completion record, receipt, reset, or deletion is
recognized rather than duplicated; a failed, incomplete, interrupted, or declined save
never permits reset or deletion. `COMPLETED.md` is the sole durable dispatch ledger.

Redact secrets everywhere and never add folders, indexes, runtime dependencies, or nested
dispatch structures.

## Git boundaries

Dispatch never performs or implies automatic commits, pushes, merges, branch deletion, or
worktree cleanup. A writer may not switch checkout or branch. If the READY receipt no longer
matches the assigned cwd, branch, SHA, or scope, block the task and return to `worktrees`
for recovery and a newly confirmed setup. Workflow approval and dispatch approval never
 authorize repository-changing Git operations.
