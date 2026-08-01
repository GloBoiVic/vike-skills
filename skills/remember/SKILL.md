---
name: remember
description: Save what matters at the end of a session so the next session picks up exactly where you left off. Or restore context at the start of a new session so nothing is lost between them.
---

AI has no memory between sessions. Every new session starts blank. This skill fixes that.

Run it at the end of a session to save. Run it at the start of a new session to restore.

## Security Boundary

This skill must never persist secrets. If any sensitive value appears in the conversation or context, do not copy it to `memory.md`.

Sensitive data includes (non-exhaustive):

- API keys, access tokens, refresh tokens, session tokens
- Passwords, passphrases, one-time codes, private keys, certificates
- Cookies, auth headers, connection strings, webhook secrets
- Any credential-like value or secret-looking string

If a detail is useful but sensitive, store a redacted placeholder instead (for example: `[REDACTED_API_KEY]`).
If unsure whether something is sensitive, treat it as sensitive and omit or redact it.

## How to Invoke

**To save at end of session:**

```
/remember save
```

**To restore at start of new session:**

```
/remember restore
```

If the developer just runs `/remember` without specifying — ask them which one they need.

---

## Save Mode

When the developer runs `/remember save`:

### What to capture

Review the current conversation to extract only what a developer would genuinely need to continue this work in a completely fresh context. Do not include sensitive data such as credentials, API keys, or tokens in the saved memory. Not a transcript. Not a summary of everything that happened. The essential state.

Capture:

**What was built** — specific files created or modified, features completed, components added. Be precise. Not "built the auth flow" — "created app/(auth)/login/page.tsx, app/(auth)/callback/page.tsx, and middleware.ts. OAuth with Google and GitHub working end to end."

To gather this information, use `git diff` against the branch point, or check file modification timestamps in relevant directories. Do not rely on your memory of the conversation — the file system is the source of truth.

**Decisions made** — choices that would be hard to reverse or that future work depends on. Not implementation details — architectural choices. "Chose to use server-side data fetching over client-side — avoids loading states and keeps sensitive logic off the client."

**Problems solved** — any issue that took time to figure out. So the next session does not solve the same problem twice. "Third party auth callback requires a trailing slash in the redirect URL — fixed in the callback handler."

**Current state** — exactly where things stand right now. What works, what is partial, what is known to be broken.

**What comes next** — the very next thing that needs to happen. Specific enough that the next session can start immediately without figuring out where to begin.

**Open questions** — anything unresolved that the next session needs to address.

### What not to capture

- Implementation details that are visible in the code
- Decisions already documented in context files — `context/index.md` and the context docs it indexes are the project source of truth; reference them, do not restate them
- Anything that can be inferred by reading the codebase
- The process of how something was built — only what was built and what was decided
- Any secrets or credential-like values (tokens, keys, passwords, cookies, auth headers, connection strings)

### What to preserve during updates

When updating an existing memory.md, do not discard:
- Eureka moments or insights that changed understanding
- Decisions that are still relevant to current work
- Problems solved that could recur
- Context that explains why things are the way they are
- Open questions that haven't been answered yet

### Safety check before writing

Before writing `memory.md`, run a final pass over the content to ensure no sensitive value is present.

- If sensitive content is found, remove or redact it before writing.
- Keep only the minimal non-sensitive context needed to continue next session.

### Where to save

Write the memory to `memory.md` in the project root. This file accumulates state across sessions — each save merges new information with existing content, preserving important context from previous sessions.

**Before saving, read `context/index.md` first** — it is the project knowledge source of truth. Check which project facts are already documented in context docs, and do not duplicate them into `memory.md`; reference the doc instead. Then read `/dispatch/COMPLETED.md`, `/dispatch/DECISIONS.md`, and `/dispatch/TASKS.md` — these are the factual record of what was built and decided. Use them as primary sources for the memory summary rather than conversation memory.

**If `memory.md` does not exist**, write the new file and confirm.

**If `memory.md` already exists**, read it first, then update:

Step 1 — Read `memory.md`, provide a brief summary, and list what the new session adds:

```
memory.md already exists.
Current: [one-line summary of existing content].
This session adds: [what's new or changed].
```

Step 2 — Ask for confirmation before updating:

```
Update memory.md with this session's state? (yes / no)
```

Step 3 — After the developer responds:

- If they say **yes**, merge the new session state into the existing content:
  - **Add** new items to "What was built", "Decisions made", "Problems solved"
  - **Replace** "Current state" and "Next session starts with" with the latest
  - **Preserve** eureka moments, insights, and context from previous sessions that still matter
  - **Keep** open questions that haven't been answered yet
  - Update the "Last updated" timestamp
- If they say **no**, do not write anything and reply:

```
No changes made. memory.md is unchanged.
```

### Format

```markdown
# Memory — [Feature or Session Name]

Last updated: [date and time]

## What was built

[Specific files, components, features completed this session]

## Decisions made

[Architectural and implementation decisions that future work depends on]

## Problems solved

[Issues resolved this session — so they are not solved again]

## Eureka moments

[Insights that changed understanding — the "aha" moments worth remembering]

## Current state

[Exactly where things stand — what works, what is partial, what is broken]

## Next session starts with

[The very first thing to do in the next session — specific and actionable]

## Open questions

[Anything unresolved that needs addressing]
```

After writing the file, confirm to the developer:

```
Memory saved to memory.md.

Next session: run /remember restore to pick up from here.
```

If this was an update (not a new file), also mention what was preserved:

```
Preserved from previous session: [what was kept].
Added from this session: [what's new].
```

---

## Restore Mode

When the developer runs `/remember restore` at the start of a new session:

### Step 1 — Find the memory

Look for `memory.md` in the project root. If it does not exist, tell the developer:

```
No memory.md found in this project.

Either this is the first session, or the file was not saved.
To save memory at the end of a session, run /remember save.
```

### Step 2 — Read everything available

Read `memory.md` first. Then read all files in /dispatch/ — PLAN.md, ARCHITECTURE.md, TASKS.md, DECISIONS.md, COMPLETED.md, MODEL-LOG.md, REVIEW.md. These contain the project's current state and decision history.

Then read `context/index.md` and treat it and the context docs it indexes as the project source of truth — load only the docs relevant to the session using the index descriptions, and do not restate their contents in memory. Only facts that live nowhere else belong in `memory.md`.

Then check for common agent context files at the project root — for example:

- `CLAUDE.md`, `.claude/context.md` — Claude Code
- `.github/copilot-instructions.md` — GitHub Copilot
- `.cursorrules`, `.cursor/rules/` — Cursor
- `.windsurfrules` — Windsurf
- `AGENTS.md` — Codex
- `.clinerules` — Cline
- `context.md` — generic fallback

This list is not exhaustive. Check for any file that looks like it establishes agent behavior rules or project conventions. Do not scan or read source code files — only these context files. Build the most complete picture possible from what is available.

When restoring, never repeat or surface raw secrets from any source. If a secret appears in restored context, summarise it in redacted form only.

### Step 3 — Confirm what was restored

Do not start building. Do not assume the developer wants to continue immediately. Summarize what was restored so the developer can verify the agent understood correctly.

```
Memory restored. Here is where we are:

**Last session:** [what was built]
**Current state:** [what works right now]
**Decisions in place:** [key decisions that are locked]
**Next up:** [what the next session should start with]

Is this correct? Say yes to continue, or correct anything
that does not look right before we proceed.
```

Only after the developer confirms does the session continue.

### If memory is incomplete or unclear

If `memory.md` exists but is missing important context, say so honestly:

```
I found memory.md but some context seems missing —
[what is unclear or absent].

Should we continue with what we have, or do you want
to fill in the gaps before we start?
```

Do not guess. Do not assume. Surface the gap and let the developer decide.
