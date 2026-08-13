---
name: dispatch
description: Execute implementation plans by dispatching fresh subagents per task with automated review loops. Each task gets an isolated implementer, a spec compliance + code quality review, and a fix loop when needed.
---

# Dispatch — Subagent-Driven Development

Execute a plan by dispatching a fresh implementer subagent per task, a task review (spec compliance + code quality) after each, and a broad whole-branch review at the end. Read-only exploration/research is a separate lane from writing.

**Why subagents:** You delegate tasks to specialized agents with isolated context. By precisely crafting their instructions and context, you ensure they stay focused and succeed at their task. They should never inherit your session's context or history — you construct exactly what they need. This also preserves your own context for coordination work.

**Core principle:** Fresh subagent per task + task review (spec + quality) + broad final review = high quality, fast iteration

**Narration:** between tool calls, narrate at most one short line — the ledger and the tool results carry the record.

**Continuous execution:** Do not pause to check in with your human partner between tasks. Execute all tasks from the plan without stopping. The only reasons to stop are: BLOCKED status you cannot resolve, ambiguity that genuinely prevents progress, or all tasks complete. "Should I continue?" prompts and progress summaries waste their time — they asked you to execute the plan, so execute it.

This does not override the mandatory explicit human confirmation before implementation on Feature, Architecture, or Security-sensitive work.

## When to Use

Use `/dispatch` when you have a written implementation plan with independent tasks. Each task should be something a fresh subagent can implement, test, and commit on its own.

Do NOT use when tasks are tightly coupled and depend on shared mutable state during implementation.

## The Process

1. **Read the plan** — understand the full scope, global constraints, and task list
2. **Create or read /dispatch/ folder** — use the flat `/dispatch/` files only. Check for existing /dispatch/PLAN.md, /dispatch/TASKS.md, /dispatch/DECISIONS.md, /dispatch/COMPLETED.md. Resume from where you left off — never re-dispatch completed tasks.
3. **Create todos** — one per task from the plan
4. **Per task:**
   a. Extract task brief from the plan (the exact requirements and code)
   b. Dispatch an implementer subagent with the brief + context + report file path
   c. Implementer builds, tests, commits, self-reviews, reports status
   d. Generate the diff/review package
   e. Dispatch a task reviewer subagent with the diff + brief + global constraints
   f. Reviewer reports spec compliance ✅/❌ and quality Approved/Issues
   g. If issues found — hand the findings back to the **same builder** who implemented the task to fix them, then re-review
   h. Mark task complete in /dispatch/TASKS.md, append to /dispatch/COMPLETED.md, log model in /dispatch/MODEL-LOG.md
5. **After all tasks** — dispatch a final whole-branch code review using the `review` skill's three-layer criteria (plan alignment, system integrity, production readiness)
6. **Present results** — summary of what was built, what was reviewed, any remaining minor issues

### Exploration lane

Run exploration serially unless the orchestrator explicitly selects bounded fan-out. Fan-out may use at most 3 read-only `explore` or `research` workers, one level deep, for independent questions only. Workers return findings; the orchestrator or documenter persists them. Do not run implementation, test, reviewer, or documenter writers in parallel. Always complete Explore → Architect → explicit human confirmation before implementation, and execute implementation writers sequentially.

## Handling Implementer Status

Implementer subagents report one of four statuses:

**DONE:** Generate the review package and dispatch the task reviewer.

**DONE_WITH_CONCERNS:** The implementer completed the work but flagged doubts. Read the concerns before proceeding. If concerns are about correctness or scope, address them before review. If they're observations, note them and proceed.

**NEEDS_CONTEXT:** Provide the missing context and re-dispatch.

**BLOCKED:** Assess the blocker. If it needs more context, provide it. If the task is too large, break it into smaller pieces. If the plan itself is wrong, escalate to the human.

**DISCOVERY:** The implementer completed the task but discovered something that changes the plan for subsequent tasks (e.g., an API does not work as documented, a dependency is unavailable, a constraint was missed). Read the discovery, update the plan for remaining tasks, then proceed.

## Handling Reviewer Findings

- **Critical / Important** findings → hand them back to the **same builder** who implemented the task to fix, then re-review
- **Minor** findings → record in the progress ledger, flag for the final review
- A finding labeled "plan-mandated" that conflicts with the plan text → present both to the human and ask which governs

## File Handoffs

Use files for all handoffs between your session and subagents:
- **Task brief:** extract the task's requirements into a file before dispatch
- **Report file:** the implementer writes their full report here; they return only status, commits, test summary, and concerns in their response
- **Review package:** the diff/commit range for the reviewer to examine
- **Progress ledger:** /dispatch/TASKS.md and /dispatch/COMPLETED.md (survive context compaction)

## Prompt Templates

### Implementer Dispatch

When dispatching an implementer, include:
1. One line on where this task fits in the project
2. The task brief path — "read this first — it is your requirements, with the exact values to use verbatim"
3. Interfaces and decisions from earlier tasks that the brief cannot know
4. Your resolution of any ambiguity you noticed in the brief
5. The report-file path and report contract (write full report there, return status + commits + test summary + concerns in response)

### Task Reviewer Dispatch

When dispatching a task reviewer, include:
1. The task brief path (same as implementer's)
2. The report file path
3. The review package path (diff/commit range)
4. Global constraints from the plan verbatim
5. Two verdicts required: spec compliance (all requirements met? nothing extra?) and task quality (code quality, test coverage, following conventions?)
6. Use the `review` skill's three-layer criteria to evaluate quality — plan alignment, system integrity, and production readiness

### Fix Loop Dispatch (same builder)

Findings go back to the **same builder** who implemented the task — they already hold the task context and know the code they wrote. Include:
1. All findings from the reviewer (Critical and Important)
2. The file paths that need changes
3. The covering test files to re-run after the fix
4. Report contract: append fix report with test results to the report file

## /dispatch/ Folder — Source of Truth

Conversation memory does not survive compaction. Track all progress in the flat `/dispatch/` folder at project root:

```
<project-root>/dispatch/
├── PLAN.md          # Current implementation plan
├── ARCHITECTURE.md  # Architecture decisions and boundaries
├── TASKS.md         # Task state (todo/in-progress/done/blocked)
├── DECISIONS.md     # Append-only decision log
├── REVIEW.md        # Review reports per feature
├── MODEL-LOG.md     # Model usage per task for budget tracking
├── EXPLORATION.md   # Exploration context (from explore agent)
└── COMPLETED.md     # Accumulated completion history
```

### Rules:
- At `/dispatch` start, read all existing files in /dispatch/ to understand current state
- Tasks listed as complete in TASKS.md or COMPLETED.md are DONE — never re-dispatch them
- After each task completes, update TASKS.md and append to COMPLETED.md
- After each decision, append to DECISIONS.md (prevents re-argument)
- After each model use, append to MODEL-LOG.md (enables budget optimization)
- The /dispatch/ folder is your recovery map after compaction
- Do not create new dispatch structures, indexes, or retention mechanisms

## Red Flags

- Do not dispatch multiple implementation subagents in parallel (causes conflicts)
- Do not dispatch explore, reviewer, or tester agents in parallel with implementation writers; bounded read-only exploration is the sole opt-in fan-out lane
- Do not ignore subagent questions — answer before letting them proceed
- Do not let implementer self-review replace actual review (both are needed)
- Do not loop fixes indefinitely — if the same Critical/Important issue survives two fix attempts, escalate to the developer
