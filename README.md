# Vike Skills

Agent skills for developers who want engineering discipline built into their AI workflow.

AI agents are powerful. They're also stateless, pattern-matching tools that will confidently build the wrong thing if you let them.

Vike Skills give your AI agent the engineering discipline it doesn't have by default — architectural thinking before you write code, structured review after, cross-session memory, systematic debugging, and codebase-wide auditing.

Ten skills. Zero bloat. Works with Claude Code, Cursor, Windsurf, Codex, Cline, OpenCode, and any agent that supports the SKILL.md format.

---

## Install

```bash
npx skills add GloBoiVic/vike-skills
```

Or clone the repo and point your agent's skills directory at it.

---

## Workflow Architecture

Vike Skills now supports a multi-agent engineering organization with an orchestrator-driven workflow.

### The Orchestrator Flow

```
Request → Orchestrator → [Explore] → [Architect] → Build →
Test → Review → Gate → Complete
```

The orchestrator manages the entire process — it never writes code. It reads project docs, creates plans in `/dispatch/`, delegates to specialist agents, tracks progress, and gates completion.

### Agent Specialization

| Agent | Role | Model |
|-------|------|-------|
| **orchestrator** | Engineering manager — plans, delegates, tracks, gates | GPT-5.6 Luna |
| **explore** | Repository intelligence — finds files, detects patterns, creates compressed context | DeepSeek V4 Flash |
| **architect** | Senior engineering decisions — system design, boundaries, technical plans | GPT-5.6 Luna |
| **build** | Primary implementation — writes production code, tests, refactors | GPT-5.2 Codex |
| **frontend** | UI implementation — design system, impeccable standards, imprint workflow | Kimi K2.5 |
| **backend** | API and database implementation | DeepSeek V4 Flash |
| **reviewer** | Quality control — plan alignment, system integrity, production readiness | DeepSeek V4 Flash |
| **tester** | Test implementation and coverage | DeepSeek V4 Flash |
| **documenter** | Documentation and session memory | DeepSeek V4 Flash |
| **general** | One-off tasks outside the orchestrator workflow | (default model) |

### Complexity Routing

- **Small task** (typo, button change) → Build → Quick Review
- **Feature task** (new page, new API) → Explore → Build → Test → Review
- **Architecture task** (multi-tenancy, auth system) → Explore → Architect → Build → Test → Security Review → Final Review

---

## Project Conventions

### `/dispatch/` — Project History

Every project task creates a structured history folder:

```
<project>/dispatch/
├── PLAN.md           # Current implementation plan
├── ARCHITECTURE.md   # System decisions and boundaries
├── TASKS.md          # Task state tracker
├── DECISIONS.md      # Append-only decision log
├── REVIEW.md         # Review reports per feature
├── MODEL-LOG.md      # Model usage for budget tracking
├── EXPLORATION.md    # Compressed context from explore agent
└── COMPLETED.md      # Accumulated completion history
```

### `/context/` — Project Knowledge Source

```
<project>/context/
├── index.md              # Manifest — agents read this first
├── project-brief.md      # Core — goals, users, scope
├── tech-stack.md         # Core — languages, frameworks, versions
├── architecture.md       # Core — system design, boundaries
├── coding-standards.md   # Core — conventions, testing
├── database.md           # Optional — schema, migrations
├── design.md             # Optional — UI/UX principles
├── ui-registry.md        # Optional — visual component registry
├── ui-tokens/            # Optional — design token files
├── library-docs.md       # Optional — key library notes
├── security.md           # Optional — auth, threats, compliance
├── api-contracts.md      # Optional — endpoint specs
└── ...
```

Run `/init` to detect, inventory, or scaffold project context. Never overwrites existing files.

---

## Skills

### Orchestration

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **orchestrate** | (loaded by orchestrator agent) | Engineering orchestrator workflow — intake, plan, delegate, track, gate. Never writes code. |
| **dispatch** | `/dispatch` | Execute plans by dispatching fresh subagents per task with review loops. Creates /dispatch/ folder. |

### Planning and Design

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **architect** | `/architect` | Think through what to build before coding. Surfaces decisions, aligns on language, produces an implementation plan. |
| **init** | `/init` | Discover or scaffold project context. Detects existing /context/, inventories files, generates starter docs when absent. Never overwrites. |

### Exploration

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **explore** | (loaded by explore agent) | Repository exploration — discovers files, detects patterns, summarizes context. Read-only. |

### Review and Quality

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **review** | `/review` | Three-layer feature review: plan alignment, system integrity, production readiness. |
| **quality** | (loaded by reviewer agent) | Five sections: debugging, failure recovery, feature review, security audit, performance audit. |
| **audit** | `/audit` | Scan codebase for security, performance, and best practice violations. Prioritized report. |

### Consistency and Memory

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **imprint** | `/imprint` | After building UI components, extract visual patterns to ui-registry.md. Flags conflicts. |
| **remember** | `/remember save` / `/remember restore` | Session memory across sessions. Reads /dispatch/ and /context/ as source of truth. |

---

## The Engineering Loop

```
/init  →  /orchestrate  (orchestrator agent)
             ↓
        /dispatch  →  [explore]  →  [architect]  →  [build/frontend/backend]
             ↓
        [tester]  →  [reviewer]  →  Gate
             ↓
        /imprint  (after every UI component)
        /remember  (end and start of every session)
```

**Budget rule:** Before using a premium model, ask: (1) Is this architecture-level? (2) Will a mistake create technical debt? (3) Is the cheaper model likely insufficient? If no to all three, use the cheaper model. Track usage in /dispatch/MODEL-LOG.md.

---

## Skill Details

### `/architect`

**Use before building anything.** Aligns on terminology (significance-based — no arbitrary count), surfaces the decisions that matter in order of impact, and produces a clear implementation plan. Includes a revision loop if the plan needs adjustment. Assumptions are labeled by confidence level — confirmed, assumed, or deferred.

### `/init`

**Use to set up project context.** If `/context/` exists, inventories files and creates `context/index.md` only if missing. If absent, asks the project type and generates core templates (`project-brief.md`, `tech-stack.md`, `architecture.md`, `coding-standards.md`) plus relevant optional docs. Never overwrites or deletes existing files.

### `/dispatch`

**Use when you have a plan to execute.** Breaks the plan into tasks, dispatches a fresh subagent per task, runs a spec compliance + code quality review after each, and loops on fixes. Uses the /dispatch/ folder for durable progress that survives session compaction.

### `/review`

**Use after building any feature.** Three layers of verification: does it match the plan? Does it respect the system architecture and code standards? Is it production-ready? Critical issues are offered for fixing immediately; Important and Minor issues wait for the developer. After fixes, only affected layers are re-reviewed.

### `/quality`

**Use for code quality operations.** Five sections covering debugging (reproduce → isolate → root cause → fix → defense in depth), failure recovery (three failure modes), feature review (three layers), security audit, and performance audit. Load only the section you need.

### `/audit`

**Use periodically or before releases.** Three-phase scan: security (secrets, OWASP top 10, auth, injection, dependencies), performance (N+1 queries, bundle size, rendering, memory), and best practices (TypeScript, error handling, testing, architecture). Scope to specific areas with `/audit security`, `/audit performance`, or `/audit practices`.

### `/imprint`

**Use after building any UI component.** Extracts the visual patterns that matter — backgrounds, borders, radii, text colors, spacing, interactive states — and saves them to ui-registry.md. Flags conflicts when a new component disagrees with existing registry entries rather than silently overwriting. Includes audit mode for establishing a baseline on existing projects.

### `/remember`

**Use at the end and start of every session.** AI has no memory between sessions. `/remember save` compresses what matters into memory.md, using /dispatch/ and /context/ files as the factual record. `/remember restore` reads it back and confirms before continuing. Includes a security boundary — never persists secrets.

---

## Credits

Vike Skills builds on excellent open-source work, adapted and extended for our own approach:

- **[JSM Agent Skills](https://github.com/JavaScript-Mastery-Pro/jsm-agent-skill)** (MIT) by JavaScript Mastery — The `/architect`, `/remember`, `/review`, `/recover`, and `/imprint` skills originated there. Their clean, opinionated skill design set the standard for how agent skills should work.
- **[Superpowers](https://github.com/obra/superpowers)** (MIT) by Jesse Vincent and Prime Radiant — The `/dispatch` skill adapts the subagent-driven-development methodology from Superpowers. The ideas of per-task subagents, automated review loops, and durable progress ledgers originate there.

Thank you to both projects for their contributions to the agent engineering community.

---

## License

MIT
