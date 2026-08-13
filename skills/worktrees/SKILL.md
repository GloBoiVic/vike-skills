---
name: worktrees
description: Opt-in guidance for planning and safely using Git worktrees; never executes worktree commands automatically.
---

# Worktrees

This skill is inert unless explicitly invoked. It provides guidance for isolating work in Git worktrees; it does not create, switch, remove, or repair worktrees on its own.

## When to use

- Use when a user explicitly asks for worktree planning or wants isolated branches for a task.
- Do not use as an automatic branch-management step or as permission to alter a repository.

## Required plan and outputs

Before proposing or executing anything, make the user confirm all of the following:

- **Path** — exact worktree directory and whether it already exists.
- **Branch** — exact branch name and starting revision/branch.
- **Scope** — repository and intended work.
- **Cleanup** — who removes the worktree/branch, when, and whether uncommitted work must be preserved.
- **Recovery** — how to locate the worktree, recover uncommitted changes, and handle an interrupted or failed command.

Return the proposed commands, their expected effects, validation checks, cleanup steps, and recovery steps before execution.

## Safety boundaries

- Human confirmation is required immediately before every worktree-affecting command, including `git worktree add`, `remove`, `move`, `prune`, branch creation, and branch deletion.
- Never guess a path, branch, revision, cleanup policy, or recovery action.
- Never discard uncommitted changes, force-reset, force-delete, or overwrite an existing path without explicit confirmation.
- Inspect status and path conflicts safely; do not install tools, use credentials, or contact remotes unless separately requested and confirmed.

## Invocation rule

Never invoke this skill automatically. Guidance alone does not authorize commands; stop until the human confirms the complete plan and each requested operation.
