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

## Quick Start

Select the orchestrator agent before starting a feature request:

```bash
# CLI syntax is version-dependent; the TUI/Tab is authoritative.
opencode agent set orchestrator     # for planning, delegating, tracking
opencode agent set build            # for direct implementation
opencode                           # no agent flag → falls back to build (no default_agent; general must be selected explicitly)
```

Once the agent is selected, work in-session:

```
/init                              # scaffold or discover project context

> Add a user settings page with theme toggle and password change

/orchestrate                       # orchestrator plans and delegates

/remember save                     # persist session state before closing
/remember restore                  # restore context next session
```

---

## Workflow Architecture

Vike Skills supports a multi-agent engineering organization with an orchestrator-driven workflow.

### The Orchestrator Flow

```
Request → Orchestrator → [Explore] → [Architect] → Build →
                                     Test → Review → Gate → Complete
```

The orchestrator manages the entire process — it never writes code. It reads project docs, creates plans in `/dispatch/`, delegates to specialist agents, tracks progress, and gates completion.

### Agent Specialization

| Agent | Role | Model |
|-------|------|-------|
| **orchestrator** | Engineering manager — plans, delegates, tracks, gates | opencode/gpt-5.6-luna |
| **explore** | Repository intelligence — finds files, detects patterns, creates compressed context | opencode/deepseek-v4-flash |
| **architect** | Senior engineering decisions — system design, boundaries, technical plans | opencode/gpt-5.6-luna |
| **build** | Primary implementation — writes production code, tests, refactors | opencode/gpt-5.2-codex |
| **frontend** | UI implementation — design system, impeccable standards, imprint workflow | opencode/kimi-k2.5 |
| **backend** | API and database implementation | opencode/deepseek-v4-pro |
| **reviewer** | Quality control — plan alignment, system integrity, production readiness | opencode/gpt-5.6-luna |
| **reviewer-premium** | Tier 3 security and high-risk architecture review | opencode/gpt-5.6-sol (manually swappable) |
| **tester** | Test implementation, coverage | opencode/deepseek-v4-flash |
| **documenter** | Documentation, session memory | opencode/deepseek-v4-flash |
| **general** | One-off tasks outside the orchestrator workflow — manual escape hatch. Must be selected explicitly via TUI/Tab or `opencode agent set general` (not the fallback agent). | (no override; uses active OpenCode default model) |

> **No custom fallback is configured.** `fallback_model` is not a supported automatic fallback field. If a configured model is unavailable, use OpenCode's current model resolution or manually switch the agent model.

### Complexity Routing

| Task Size | Flow |
|-----------|------|
| **Small** (typo, button change, comment fix) | Build → Quick Review (Tier 1) |
| **Feature** (new page, new API, new component) | Explore → Build → Test → Review (Tier 2) |
| **Architecture** (multi-tenancy, system design, refactoring) | Explore → Architect → Build → Test → Review (Tier 2 — Luna reviewer) |
| **Security-sensitive** (auth, payments, security redesign) | Explore → Architect → Build → Test → `reviewer-premium` (Tier 3) |

### Review Tiers

- **Tier 1 — Skip formal review.** For docs, styling, simple fixes, and trivial changes. No reviewer agent is dispatched.
- **Tier 2 — Luna reviewer.** Full formal review for features, API changes, database schema, and architecture decisions. Uses the **reviewer** agent (opencode/gpt-5.6-luna) to verify plan alignment, system integrity, and production readiness.
- **Tier 3 — Premium review.** Uses the explicit `reviewer-premium` agent for authentication, payments, security, and major security-sensitive redesigns. Its model is manually swappable in `opencode.jsonc`.

### General Agent (Escape Hatch)

The **general** agent has no model override — it uses whatever model OpenCode is running by default. **It is not the fallback agent.** Running `opencode` with no agent flag falls back to the `build` agent (the config has no `default_agent`). To use the general agent, select it explicitly via the TUI/Tab or `opencode agent set general`. Use it for one-off tasks that don't fit the orchestrator flow: quick inspections, ad-hoc scripts, exploratory queries, or anything that doesn't need planning, delegation, and review overhead.

---

## Project Conventions

### `/dispatch/` — Source of Truth for History

Every project task creates a structured history folder. `/dispatch/` files are the **definitive factual record** — agents read and write here, never relying on conversation history for decisions.

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

### `/context/` — Source of Truth for Knowledge

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

### Frontend Design

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **impeccable** | (loaded by frontend agent) | Design, redesign, critique, polish, and optimize frontend interfaces. Covers UX review, visual hierarchy, accessibility, responsive behavior, typography, color, motion, micro-interactions, and design systems. |
| **imprint** | `/imprint` | After building UI components, extract visual patterns to ui-registry.md. Flags conflicts when a new component disagrees with existing registry entries. |
| **ui-ux-pro-max** | (searchable reference) | UI/UX design intelligence database with 84 styles, 192 palettes, 74 font pairings, 192 product types, 98 UX guidelines, 104 icons, 16 GSAP motion presets, and 25 chart types across 22 stacks. |

### Review and Quality

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **review** | `/review` | Three-layer feature review: plan alignment, system integrity, production readiness. |
| **quality** | (loaded for troubleshooting) | Debugging and failure recovery. Use `review` for feature reviews and `audit` for systemic audits. |
| **audit** | `/audit` | Scan codebase for security, performance, and best practice violations. Prioritized report. |

### Consistency and Memory

| Skill | Invocation | Purpose |
|-------|-----------|---------|
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

## In-Session Examples

### Starting a New Feature

```
# 1. Pick your orchestrator agent
opencode agent set orchestrator

# 2. Initialize project context (one-time)
/init

# 3. Request the feature
> Add a paginated blog with category filtering and search

# Orchestrator explores, architects, dispatches build tasks, runs tests,
# reviews the result, and gates completion.

# 4. After UI work, imprint visual patterns
/imprint

# 5. End of session — save state
/remember save
```

### Quick Fix Without Orchestrator

```
# Use build agent for a direct fix
opencode agent set build

> Fix the broken link on the homepage hero button

# Simple changes skip formal Tier 1 review.
```

### One-Off Query (General)

```
# Select the general agent explicitly (not the fallback)
opencode agent set general

opencode

> How many API routes do we have, and which ones are missing auth middleware?
```

### Resume Next Session

```
opencode agent set orchestrator
/remember restore
# Work resumes exactly where it left off.
```

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

### Impeccable (Frontend Design)

**Use for any UI work.** The frontend agent loads the **impeccable** skill to design, shape, critique, polish, and optimize interfaces. Covers UX review, visual hierarchy, information architecture, cognitive load, accessibility, responsive behavior, theming, typography, color, motion, micro-interactions, error states, and design-system extraction. Handles both ambitious visual effects and conservative refinement.

### `/quality`

**Use for debugging and recovery.** Diagnose isolated bugs, polluted sessions, and wrong foundations. Use `/review` for feature review and `/audit` for systemic security, performance, and practice audits.

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
