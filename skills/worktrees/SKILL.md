---
name: worktrees
description: Opt-in plan for safely isolating Git work in a worktree; requires complete operation details and confirmation immediately before every repository-changing command.
---

# Worktrees

Use for Feature, Architecture, or Security-sensitive work before any writer runs,
and when the user explicitly requests isolation. Small work stays in the current
checkout unless the user requests a worktree. This skill plans isolation; it never
silently performs Git operations.

## Required plan

Before proposing or executing, specify and obtain confirmation for:

- **Path** — exact worktree directory and whether it exists.
- **Branch/revision** — exact branch and starting revision/branch.
- **Scope** — repository and intended work.
- **Cleanup** — owner, timing, branch handling, and preservation of uncommitted work;
  no automatic cleanup, commit, push, merge, or branch deletion.
- **Recovery** — locating the worktree, recovering changes, and handling interruption/failure.

Return commands, expected effects, validation checks, cleanup, and recovery steps.
The plan must identify a dedicated local feature branch in a linked worktree and the
exact cwd writers will use. A successful setup returns a **READY** receipt containing:
`root`, `path`, `branch`, full `SHA`, `scope`, `status`, and `recovery`.

## Safety

Confirm the exact command immediately before every Git operation, including status,
rev-parse, worktree add|remove|move|prune, branch creation, and branch deletion.
Each command needs its own confirmation; workflow or blueprint approval is not Git
authorization. Never guess parameters, discard uncommitted changes, force-reset/delete,
overwrite an existing path, install tools, use credentials, or contact remotes without
separate explicit authorization. Do not commit, push, merge, or clean up automatically.
Guidance does not authorize commands. If setup is incomplete or validation fails,
do not issue READY; stop with recovery instructions.
