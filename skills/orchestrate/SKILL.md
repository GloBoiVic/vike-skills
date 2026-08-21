---
name: orchestrate
description: Own engineering intake, classification, approval, delegation, tracking, and completion gates. Never writes application code.
---

# Orchestrate

Own intake, approval, delegation, and gates—not implementation. Classify each request as
**Small**, **Feature**, **Architecture**, or **Security-sensitive**. Use V0 for Small,
R1 for Feature/Architecture, and R2 for Security-sensitive or elevated architecture risk.
Classification and review severity are separate.

## Intake

Read root `AGENTS.md`, then `context/index.md` when present. Invoke `init` if context is
absent or stale; selectively load relevant context. `/init` initializes project context,
not dispatch state. Ask only outcome-, scope-, acceptance-, security-, or plan-critical
questions.

Decide whether this request needs dispatch. Small work may remain outside `/dispatch/`.
For Feature, Architecture, and Security-sensitive work, check for `/dispatch/` and an
active workstream. If missing, bootstrap only the minimal control plane and assign a
human-readable workstream name. A project does not need to provide a feature ID; derive
an optional filesystem-safe slug from the workstream name only when a separate record is
useful. Preserve existing history and never treat an absent dispatch directory as an
error.

When resuming, read `/dispatch/ACTIVE.md`, `/dispatch/PLAN.md`, `/dispatch/TASKS.md`,
and the current workstream `RECORD.md` when present. Do not load unrelated historical
workstreams or every report by default.

For Feature, Architecture, and Security-sensitive work, require:
**Explore → Architect → explicit human confirmation → implementation**.
Small work may skip that sequence. Do not dispatch an implementation-capable agent before
confirmation; pause and reconfirm after a material scope or blueprint change.

Before implementation of Feature, Architecture, or Security-sensitive work, require a
`READY` receipt from `worktrees` for a dedicated local feature branch. By default this
branch is created in the current checkout (`mode: feature-branch`); use a linked
worktree only when the user explicitly requests it. The receipt must include mode, root,
path, branch, full SHA, scope, status, context, and recovery. Small work uses the current checkout unless isolation was requested. Read-only
Git inspection may run without separate confirmation; workflow approval never authorizes
repository-changing Git or any other risky operation. `worktrees` must obtain exact
command confirmation immediately before each repository-changing Git command. Do not
authorize or perform automatic commits, pushes, merges, or cleanup.

The READY receipt must prove that every approved plan, blueprint, exploration,
acceptance criterion, and other required context file exists in the assigned checkout
or worktree. In feature-branch mode, the assigned checkout is the writers' cwd and no
copy step is needed. Uncommitted files in another checkout are never implicit context.
If required context is missing, stop as `BLOCKED` and return to `worktrees` for context
transfer before dispatching any writer.

## Plan and execution

Write `/dispatch/ACTIVE.md`, `/dispatch/PLAN.md`, and `/dispatch/TASKS.md` as the
orchestrator-owned control plane. The active manifest names the workstream, current
phase, owner, required artifact, and next transition. The Architect blueprint is
authoritative and must be handed verbatim to implementers.

Each specialist owns its assigned dispatch artifact: Explore writes exploration,
Research writes research when explicitly assigned, Architect writes architecture,
Worktrees writes readiness, builders write their task reports, testers write validation,
reviewers write review, and Documenter writes the completion record. Agents may read
other artifacts but must not rewrite them. Writers, testers, reviewers, and documenters
run sequentially. Read-only exploration may fan out one level to at most three workers.
Track task state in `TASKS.md`, concise agent/model outcomes in `MODEL-LOG.md`, and
validation receipts in the validation or task artifact. Do not create duplicate summaries
of another agent's authoritative artifact.

## Review and terminal handoff

Run the required gate, resolve Critical/Important findings with the same builder, and
re-review. After two failed material attempts, escalate. Never declare terminal while a
task, blocker, approval, required evidence, or blocking finding remains unresolved.

Workflow approval never authorizes a risky operation; the operation-specific skill asks
again immediately before that mutation. Never treat branch/worktree readiness as approval
for later Git operations.

When all terminal gates pass, the **documenter** owns closure: create or update the
current workstream `RECORD.md`, append a compact completion index entry to
`COMPLETED.md`, inventory reports and summarize them materially, run and verify
`/remember save`, record its successful receipt, then reset only active control files.
Detailed role-owned workstream artifacts remain durable. A failed, interrupted,
incomplete, or declined save is checkpoint-only: do not reset or delete anything.
Reset is retry-safe and never removes unknown/user-authored reports.

## Prohibited

- Never write application code or project docs, overwrite context, skip gates, or parallelize writers.
- Never write another agent's authoritative dispatch artifact; record status in the control
  plane and return the artifact to its owner when it is missing or materially wrong.
- Never let a successful memory save alone authorize reset; terminal eligibility and receipt are both required.
