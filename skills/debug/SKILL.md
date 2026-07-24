---
name: debug
description: Systematic root-cause debugging. Four-phase process — reproduce and isolate the failure, find the root cause, apply a precise fix with verification, and prevent recurrence.
---

Not every bug is obvious. Not every fix is correct on the first try.

This skill replaces guessing with a systematic process. Four phases, followed in order. Each phase has a clear output before the next one begins.

## How to Invoke

```
/debug
```

Describe what is broken. The skill takes it from there.

---

## Phase 1 — Reproduce and Isolate

Before any fix attempt, establish what is actually happening.

### Step 1.1 — Reproduce it yourself first

Attempt to reproduce the failure yourself. Run the relevant code, check the error, confirm you can see the wrong behaviour that was described.

If you have access to error logs, stack traces, or test output — start there before asking any questions.

### Step 1.2 — Ask the developer if you cannot reproduce

Only ask the developer after you have tried and failed to reproduce:

- What did you expect to happen?
- What happened instead?
- What are the exact steps to reproduce?
- Any error messages, stack traces, or console output?

If you can reproduce the bug without asking, skip this step entirely.

### Step 1.3 — Isolate to a specific location

Narrow the failure to a specific file, function, or component. Use binary search if needed — comment out half the suspected code, see if the error persists, repeat.

State the isolation result:

```
Isolated to: [file:line] — [function/component name]
Evidence: [what confirms this is the location]
```

Do not proceed to Phase 2 until the failure is isolated to a specific location you can point to.

---

## Phase 2 — Root Cause Analysis

Find the actual cause of the failure — not a symptom, not a related issue, the actual cause.

### Step 2.1 — Read the code at the isolated location

Read the function or component where the failure occurs. Understand what it is supposed to do and what it actually does.

### Step 2.2 — Trace the data flow

Follow the data through the code path:

- What inputs does this code receive?
- What transformations happen to those inputs?
- What outputs are produced?
- Where does the actual output diverge from the expected output?

### Step 2.3 — Identify the root cause

The root cause is the specific line, condition, or assumption that produces the wrong output. It is not "the database returned bad data" — it is "the query filter uses `userId` when it should use `organizationId`."

State the root cause clearly:

```
Root cause: [specific, actionable statement of what is wrong]

How it was found: [brief explanation of the analysis path]

Why this is the root cause and not a symptom:
[explanation of the cause-effect chain]
```

Do not proceed to Phase 3 until the root cause is stated as a specific, actionable statement.

---

## Phase 3 — Fix and Verify

Apply a precise fix that addresses the root cause, then verify it works.

### Step 3.1 — Design the fix

Describe the fix before writing any code:

```
Fix: [what needs to change]

This fixes the root cause because:
[how the change addresses the specific root cause identified in Phase 2]
```

Present this to the developer. Wait for confirmation before writing code.

### Step 3.2 — Apply the fix

Make the minimal change needed. Do not fix unrelated issues during this step — one fix, one purpose.

### Step 3.3 — Verify the fix

- Run the reproduction steps from Phase 1 — confirm the failure no longer occurs
- Run existing tests that cover this code — confirm nothing is broken
- If no test covers this code, run the fix through at least one manual verification

### Step 3.4 — Confirm with the developer

```
Fix applied at [file:line].

Verified:
- [X] Failure no longer reproduces
- [X] Existing tests pass ([N]/[N])
- [X] Manual verification confirms correct behaviour

Does this resolve the issue from your perspective?
```

---

## Phase 4 — Defense in Depth

The bug is fixed. Now ensure it stays fixed and does not reappear elsewhere.

### Step 4.1 — Add a regression test

If no test exists for this behaviour, add one that would catch this bug if it reappears. The test should:

- Exercise the exact scenario that was broken
- Assert the correct behaviour
- Fail if the bug is reintroduced

### Step 4.2 — Scan for similar patterns

Check if the same mistake exists elsewhere in the codebase:

- Same pattern used in other files?
- Same misconception in other parts of the system?
- Same type of bug in related code?

Report any findings:

```
Related patterns found:
- [file:line] — [same root cause pattern present here]
- [file:line] — [same root cause pattern present here]

Recommendation: [fix / monitor / no action needed — with reasoning]
```

### Step 4.3 — Reflect on prevention

Identify what allowed this bug to exist:

- Was the requirement unclear?
- Was the interface misleading?
- Was there insufficient validation?
- Was there no test for this scenario?

```
Prevention note: [what would prevent this class of bug in the future]

Action: [specific recommendation — add validation, clarify docs, 
add tests, etc.]
```

---

## When to Stop

If at any point you cannot make progress:

- **Phase 1** — cannot reproduce or isolate → ask the developer for more detail
- **Phase 2** — cannot find root cause after reasonable effort → escalate with what you know
- **Phase 3** — fix does not work → return to Phase 2 (root cause may be wrong)
- **Phase 3** — if two root cause attempts both fail → may be a deeper problem. Use the `recover` skill to diagnose whether this is a Failure Mode 2 (polluted session) or Failure Mode 3 (wrong foundation).
- **Phase 4** — scale effort to the bug. For trivial bugs (typo, wrong variable name, one-character fix), skip Phase 4 unless the developer explicitly asks. For complex bugs, complete all three steps.

---

## Red Flags

- Do not guess at root causes — trace the actual data flow
- Do not skip the regression test for complex bugs