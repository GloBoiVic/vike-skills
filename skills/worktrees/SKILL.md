---
name: worktrees
description: Opt-in plan for safely isolating Git work in a worktree; requires complete operation details and confirmation immediately before every repository-changing command.
---

# Worktrees

Use only when the user explicitly requests worktree planning or isolated branches. It is not automatic branch management and never creates, switches, removes, prunes, moves, or repairs worktrees on its own.

## Required plan

Before proposing or executing, specify and obtain confirmation for:

- **Path** — exact worktree directory and whether it exists.
- **Branch/revision** — exact branch and starting revision/branch.
- **Scope** — repository and intended work.
- **Cleanup** — owner, timing, branch handling, and preservation of uncommitted work.
- **Recovery** — locating the worktree, recovering changes, and handling interruption/failure.

Return commands, expected effects, validation checks, cleanup, and recovery steps.

## Safety

Confirm immediately before every worktree-affecting command, including `git worktree add|remove|move|prune`, branch creation, and branch deletion. Never guess parameters, discard uncommitted changes, force-reset/delete, overwrite an existing path, install tools, use credentials, or contact remotes without separate explicit authorization. Guidance does not authorize commands; stop until the complete plan and each requested operation are confirmed.
