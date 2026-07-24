# Vike Skills

Agent skills for developers who want engineering discipline built into their AI workflow.

AI agents are powerful. They're also stateless, pattern-matching tools that will confidently build the wrong thing if you let them.

Vike Skills give your AI agent the engineering discipline it doesn't have by default — architectural thinking before you write code, structured review after, cross-session memory, systematic debugging, and codebase-wide auditing.

Eight skills. Zero bloat. Works with Claude Code, Cursor, Windsurf, Codex, Cline, OpenCode, and any agent that supports the SKILL.md format.

---

## Install

```bash
npx skills add GloBoiVic/vike-skills
```

Or clone the repo and point your agent's skills directory at it.

---

## Skills

### Planning and Design

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **architect** | `/architect` | Think through what to build before coding. Surfaces decisions, aligns on language, produces an implementation plan you confirm before anything starts. Not a grilling session — a thinking session. |
| **dispatch** | `/dispatch` | Execute implementation plans by dispatching fresh subagents per task with automated review loops. Each task gets an isolated implementer, spec compliance + code quality review, and a fix loop when needed. |

### Review and Quality

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **review** | `/review` | After building a feature, verify it matches the plan, respects the system architecture, and is production-ready. Three layers: plan alignment, system integrity, production readiness. |
| **audit** | `/audit` | Scan the full codebase for security vulnerabilities, performance issues, and best practice violations. Prioritized report with severity levels — the developer decides what to fix. |

### Recovery and Debugging

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **recover** | `/recover` | When something goes wrong, diagnose the failure mode before responding. Three responses: targeted fix (isolated bug), hard reset (polluted session), or rethink (wrong foundation). |
| **debug** | `/debug` | Systematic root-cause debugging. Four phases: reproduce and isolate, find the root cause, apply a precise fix with verification, and prevent recurrence. |

### Consistency and Memory

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **imprint** | `/imprint` | After building any UI component, extract visual patterns to ui-registry.md. Flags conflicts rather than silently overwriting. Includes audit mode for existing projects. |
| **remember** | `/remember save` / `/remember restore` | Save session context at the end of a session, restore it at the start of the next. No more starting from zero every time. |

---

## The Engineering Loop

```
/architect  →  /dispatch  →  /review  →  Ship
                                  ↓
                    /imprint  (after every UI component)
                    /remember  (end and start of every session)

When something breaks:  /recover  or  /debug
Periodic health check:  /audit
```

---

## Skill Details

### `/architect`

**Use before building anything.** Aligns on terminology (significance-based — no arbitrary count), surfaces the decisions that matter in order of impact, and produces a clear implementation plan. Includes a revision loop if the plan needs adjustment. Assumptions are labeled by confidence level — confirmed, assumed, or deferred.

### `/dispatch`

**Use when you have a plan to execute.** Breaks the plan into tasks, dispatches a fresh subagent per task, runs a spec compliance + code quality review after each (using `/review`'s three-layer criteria), and loops on fixes. Includes a DISCOVERY status for when an implementer finds something that changes the plan. File-based handoffs keep your context clean. Progress ledger survives session compaction.

### `/review`

**Use after building any feature.** Three layers of verification: does it match the plan? Does it respect the system architecture and code standards? Is it production-ready? Critical issues are offered for fixing immediately; Important and Minor issues wait for the developer. After fixes, only affected layers are re-reviewed.

### `/audit`

**Use periodically or before releases.** Three-phase scan: security (secrets, OWASP top 10, auth, injection, dependencies), performance (N+1 queries, bundle size, rendering, memory), and best practices (TypeScript, error handling, testing, architecture). Uses targeted scanning strategies (regex for secrets, lock files for dependencies) rather than reading every file. False positives are labeled with uncertainty rather than asserted as findings. Scope to specific areas with `/audit security`, `/audit performance`, or `/audit practices`.

### `/recover`

**Use when something goes wrong.** Diagnoses the failure mode before responding: targeted fix (isolated problem), hard reset (polluted session — abandon and restart), or rethink (wrong foundation — the whole approach is incorrect). Concrete detection triggers for mode 3 (wrong library, wrong paradigm, contradicts project docs). After resolution, hands off to the right next skill — `/architect` after a rethink, `/remember restore` after a hard reset.

### `/debug`

**Use when a bug exists.** Four-phase systematic process: reproduce first (ask the developer only if you can't), isolate to a specific file/function, trace the data flow to find the root cause, apply a precise fix, then add regression tests. Scales Phase 4 to the bug — trivial fixes skip the deep dive. Cross-references `/recover` when multiple root cause attempts fail.

### `/imprint`

**Use after building any UI component.** Extracts the visual patterns that matter — backgrounds, borders, radii, text colors, spacing, interactive states — and saves them to ui-registry.md. Flags conflicts when a new component disagrees with existing registry entries rather than silently overwriting. Includes audit mode for establishing a baseline on existing projects, with the option to apply fixes afterward.

### `/remember`

**Use at the end and start of every session.** AI has no memory between sessions. `/remember save` compresses what matters into memory.md, gathering file changes via git diff rather than conversation memory. Asks for direction when the session's work is unrelated to previous memory. `/remember restore` reads it back and confirms before continuing. Includes a security boundary — never persists secrets.

---

## Credits

Vike Skills builds on excellent open-source work, adapted and extended for our own approach:

- **[JSM Agent Skills](https://github.com/JavaScript-Mastery-Pro/jsm-agent-skill)** (MIT) by JavaScript Mastery — The `/architect`, `/remember`, `/review`, `/recover`, and `/imprint` skills originated there. Their clean, opinionated skill design set the standard for how agent skills should work. We've revised the process flows, tightened redundancies, and added cross-skill handoffs.
- **[Superpowers](https://github.com/obra/superpowers)** (MIT) by Jesse Vincent and Prime Radiant — The `/dispatch` skill adapts the subagent-driven-development methodology from Superpowers. The ideas of per-task subagents, automated review loops, and durable progress ledgers originate there.

Thank you to both projects for their contributions to the agent engineering community.

---

## License

MIT