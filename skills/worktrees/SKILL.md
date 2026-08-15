---
name: worktrees
description: Plan safe Git isolation using a local feature branch by default, with linked worktrees as an explicit opt-in; requires confirmation immediately before repository-changing commands.
---

# Git isolation

Use for Feature, Architecture, or Security-sensitive work before any writer runs,
and when the user explicitly requests isolation. The default is a dedicated local
feature branch in the current checkout. A linked worktree is an explicit opt-in for
parallel checkouts or when the user asks for one. Small work stays on the current
branch unless the user requests isolation. This skill plans isolation; it never
silently performs Git operations.

## Default mode: feature branch

The normal workflow is intentionally simple:

1. Inspect the current repository and confirm the starting branch/SHA.
2. Propose one exact branch command, normally `git switch -c feature/<short-name>`.
3. Obtain confirmation immediately before running that command.
4. Verify the branch and use the same repository path as the writers' `cwd`.
5. Return `READY` with `mode: feature-branch`.

The user remains responsible for committing, merging, pushing, and deleting the
branch. Agents do not switch branches after `READY`.

## Core model

A linked worktree is an independent checkout, not a live view of another checkout.
Each worktree has its own branch, working tree, index, and uncommitted files. Changes
that are uncommitted in one worktree are not visible in another worktree. The worktree
directory named in the READY receipt is the writer's complete source of truth.

Never dispatch a writer to a worktree that lacks the approved plan, blueprint,
exploration, acceptance criteria, or other required context. Do not assume that files
present in the primary checkout are present in the linked worktree.

## Required plan

Before proposing or executing, specify and obtain confirmation for:

- **Mode** — `feature-branch` (default, current checkout) or `linked-worktree` (opt-in).

- **Path** — exact repository directory for `feature-branch`, or exact worktree
  directory for `linked-worktree`, and whether it exists.
- **Branch/revision** — exact branch and starting revision/branch.
- **Scope** — repository and intended work.
- **Cleanup** — owner, timing, branch handling, and preservation of uncommitted work;
  no automatic cleanup, commit, push, merge, or branch deletion.
- **Recovery** — locating the worktree, recovering changes, and handling interruption/failure.

- **Context transfer** — required for `linked-worktree`; not required for
  `feature-branch` because writers use the same checkout. The exact required
  planning/context files must still be listed.

Return commands, expected effects, validation checks, cleanup, and recovery steps.
The plan must identify a dedicated local feature branch and the exact cwd writers will
use. A successful setup returns a **READY** receipt containing:
`mode`, `root`, `path`, `branch`, full `SHA`, `scope`, `status`, `context`, and `recovery`.

## Context transfer policy (linked-worktree mode)

Before creating or approving a worktree, classify every required context file as one of:

1. **Committed baseline** — already present at the starting SHA; no transfer required.
2. **Committed task context** — commit the approved planning files to the starting
   branch before creating the worktree, then verify they exist at the worktree SHA.
3. **Explicitly copied context** — copy only the listed, non-secret files into the
   worktree after creation, with the exact source and destination recorded. This is
   preferred only when committing planning state is undesirable.
4. **Generated or external context** — recreate it using a documented deterministic
   step, or provide its absolute path explicitly if the tool supports additional roots.

Do not use uncommitted files from another checkout as implicit context. Do not copy
`.env` files, credentials, private keys, or unknown files. If required context cannot
be transferred safely, stop with `BLOCKED` rather than issuing READY.

The context manifest must list each required file and its status:

```text
context:
  - source: /absolute/source/path
    destination: /absolute/worktree/path
    method: committed-baseline | committed-task-context | explicit-copy | generated
    verified: true
```

## READY gate

Before returning READY, verify from the assigned worktree cwd that:

- the expected branch and starting SHA are correct;
- every required context file exists and is readable;
- the plan and blueprint are the approved versions;
- the worktree is the exact cwd passed to every writer;
- no required context is relying on uncommitted files in another checkout.

In `feature-branch` mode, verify the same branch/SHA and that the assigned `cwd` is
the user's current repository checkout. The context manifest may mark project files
as `committed-baseline`; no copying or second checkout is needed.

If any check fails, return `BLOCKED` with the missing files and the precise recovery
action. Do not dispatch implementation, testing, review, or documentation agents until
the gate passes.

## Safety

Read-only Git inspection commands such as `git status`, `git rev-parse`,
`git branch --show-current`, and `git diff --stat` may run without a separate
operation confirmation.

Confirm the exact command immediately before every repository-changing Git operation,
including `git worktree add|remove|move|prune`, branch creation, branch deletion,
checkout, reset, commit, push, merge, and cleanup. Each repository-changing command
needs its own confirmation; workflow or blueprint approval is not Git authorization.

Never guess parameters, discard uncommitted changes, force-reset/delete, overwrite an
existing path, install tools, use credentials, or contact remotes without separate
explicit authorization. Do not commit, push, merge, or clean up automatically.
Guidance does not authorize commands. If setup is incomplete or validation fails,
do not issue READY; stop with recovery instructions.
