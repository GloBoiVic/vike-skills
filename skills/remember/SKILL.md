---
name: remember
description: Save what matters at the end of a session so the next session picks up exactly where you left off. Or restore context at the start of a new session so nothing is lost between them.
---

# Remember — Secret-Safe Session State

Use `/remember save` at session end and `/remember restore` at session start. With no mode,
ask which is wanted. Save is a memory operation, not dispatch cleanup.

## Security boundary

Never persist or surface secrets: keys, tokens, passwords, codes, private keys, certificates, cookies, auth headers, connection strings, webhook secrets, or anything uncertain. Omit or replace useful sensitive details with placeholders such as `[REDACTED_TOKEN]`.

## Save

On `/remember save`, read `context/index.md` when present, then use the filesystem as source
of truth. Consult relevant context and, when present, `/dispatch/COMPLETED.md`,
`DECISIONS.md`, and `TASKS.md`. Do not duplicate facts already in context.

Capture only what a fresh session needs: files/features built; durable decisions and problems solved; exact current state; next action; and open questions. Do not save a transcript, inferable implementation details, process history, or secrets. Redact again immediately before writing.

Write or merge root `memory.md`. If it exists, read it and report its current state plus this session's additions, then ask:

`Update memory.md with this session's state? (yes / no)`

On **no**, the save is declined: write nothing and return a non-success receipt. On **yes**,
merge built/decision/problem items, replace current state and next action, preserve relevant
insights/open questions, and update the timestamp. Report a successful receipt only after
`memory.md` is durably written. Dispatch reset and report cleanup belong to the documenter
closure sequence, not this save operation.

```markdown
# Memory — [Feature or Session Name]
Last updated: [date and time]
## What was built
## Decisions made
## Problems solved
## Eureka moments
## Current state
## Next session starts with
## Open questions
```

Confirm `Memory saved to memory.md.` and advise `/remember restore`; for updates, say what
was preserved and added. A failed, interrupted, incomplete, or declined save is never a
terminal cleanup authorization. The documenter must record a successful receipt in
`/dispatch/COMPLETED.md` before any terminal reset or report deletion.

## Restore

1. Find root `memory.md`; if absent, say so and suggest `/remember save`.
2. Read it first, then relevant available dispatch records and `context/index.md`; selectively load indexed context docs. Check root agent-instruction files, but do not scan source code.
3. Redact sensitive information before summarizing.
4. Do not start work. Report the last session, current state, decisions, and next action, then ask the developer to confirm. If memory is incomplete, state what is missing and ask whether to continue; never guess.

```text
Memory restored. Here is where we are:
Last session: [what was built]
Current state: [what works/does not]
Decisions in place: [key decisions]
Next up: [first action]
Is this correct? Say yes to continue, or correct anything that does not look right.
```
