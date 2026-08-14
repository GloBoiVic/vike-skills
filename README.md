# Vike Skills

Agent skills for developers who want engineering discipline built into their AI workflow.

AI agents are powerful, stateless, and pattern-matching — they will confidently build the wrong thing without structure. Vike Skills add the discipline: architectural thinking before code, task classification with matched review gates, structured review after, cross-session memory, systematic debugging, and codebase-wide auditing.

**15 lean skills** in this repo, plus optional frontend reference skills (`impeccable`, `ui-ux-pro-max`) from your agent library. Works with any agent that supports the SKILL.md format — Claude Code, Cursor, Windsurf, Codex, Cline, OpenCode.

## Install

```bash
npx skills add GloBoiVic/vike-skills
```

Or clone the repo and point your agent's skills directory at it. In OpenCode, add the path to `skills.paths` in `opencode.jsonc`.

## Policy Matrix

### Task classes and review gates

| Class | Examples | Workflow | Gate |
|-------|----------|----------|------|
| **Small** | typo, button change, comment fix | Direct implementation | V0 self-check |
| **Feature** | new page, new API, new component | Explore → Architect → human confirmation → build → test | R1 formal review |
| **Architecture** | system redesign, cross-cutting refactor | Explore → Architect → human confirmation → build → test | R1 (elevate to R2 when risk warrants) |
| **Security-sensitive** | auth, authorization, payments, secrets, security redesign | Explore → Architect → human confirmation → build → test | R2 premium/security review |

Classification is not review severity. Feature, Architecture, and Security-sensitive work must pass explicit human confirmation of the Architect's blueprint before any implementation begins and must receive a `READY` receipt from `worktrees` for a dedicated local feature branch in a linked worktree. The receipt records root, path, branch, full SHA, scope, status, and recovery. Small work stays in the current checkout unless isolation is requested. Gates: **V0** is a Small-task self-check, **R1** is formal review for Feature and Architecture, **R2** is premium/security review for Security-sensitive work or explicitly elevated architecture risk. Critical/Important findings return to the same builder for fixes and re-review; after two failed material fix attempts, escalate to the developer. Workflow approval never authorizes Git operations.

### Policy ownership

| Policy | Owner |
|--------|-------|
| Classification and workflow approval | `orchestrate` |
| Design blueprint | `architect` |
| Sequential execution, ledgers, fix loop | `dispatch` |
| Review method, severity, completion gates | `review` |
| Context bootstrap/inventory | `init` |
| Optional UI pattern capture | `imprint` |
| Secret-safe session memory | `remember` |
| Debugging recovery | `quality` |
| Systemic audits | `audit` |
| Risky-operation approval | The operation-specific skill |

### Execution rules

- Writers, testers, reviewers, and documenters run sequentially. Only read-only exploration/research may fan out — bounded, one level, at most three workers.
- All state lives in the flat `/dispatch/` set: `PLAN.md`, `ARCHITECTURE.md`, `TASKS.md`, `DECISIONS.md`, `REVIEW.md`, `MODEL-LOG.md`, `EXPLORATION.md`, `COMPLETED.md`. No nested structures, indexes, or runtime dependencies.
- Execution continues without check-ins until blocked, a material scope/design change, an operation-level confirmation, an unresolved review conflict, two failed fix attempts, or user interruption.
- Workflow approval never authorizes a risky operation. The operation-specific skill confirms immediately before each risky mutation.
- Git isolation is operation-specific: confirm the exact Git command immediately before each command. There are no automatic commits, pushes, merges, or worktree cleanup operations.
- Secrets are redacted from prompts, reports, and ledgers. `remember` never persists secrets and never resets or deletes dispatch state.
- `imprint` is optional: it runs only when requested and asks before creating a missing UI registry. There is no required or canonical registry path, no migration policy, and no `init` dependency.
- `/init` never overwrites or deletes existing files.
- Context7 is not configured.

### Agents and models

Assignments live in `opencode.jsonc`, which is the source of truth; swap models there manually. The `explore` and `documenter` agents use `openai/gpt-5.6-luna-fast`; the exact free DeepSeek identifier remains `opencode/deepseek-v4-flash-free` for reviewer and tester assignments.

| Agent | Role | Model |
|-------|------|-------|
| **orchestrator** | Engineering manager — plans, delegates, tracks, gates | openai/gpt-5.6-terra (medium) |
| **explore** | Repository intelligence — files, patterns, compressed context | openai/gpt-5.6-luna-fast |
| **architect** | Senior engineering decisions — system design, boundaries, plans | openai/gpt-5.6-sol (high) |
| **research** | Read-only evidence gathering | openai/gpt-5.6-luna |
| **build** | Primary implementation — code, tests, refactors | openai/gpt-5.6-luna |
| **frontend** | UI implementation — design system, impeccable standards | openai/gpt-5.6-luna |
| **backend** | API and database implementation | openai/gpt-5.6-luna |
| **reviewer** | Formal review (R1) — plan alignment, integrity, readiness | opencode/deepseek-v4-flash-free |
| **reviewer-premium** | Premium/security review (R2) | openai/gpt-5.6-terra (high) |
| **tester** | Test implementation, coverage | opencode/deepseek-v4-flash-free |
| **documenter** | Documentation, session memory | openai/gpt-5.6-luna-fast |
| **general** | One-off tasks — escape hatch, selected explicitly | no override (active default model) |

No `default_agent` is configured: running `opencode` with no agent flag falls back to **build**. The **general** agent is not the fallback — select it explicitly.

## Skills

### Core

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **orchestrate** | orchestrator agent | Intake, classification, approval, delegation, tracking, completion gates. Never writes code. |
| **dispatch** | `/dispatch` | Execute an approved plan: fresh implementer per task, sequential writers, review gates, ledgers, bounded fix loops. |
| **architect** | architect agent | Authoritative implementation blueprint for non-small work, confirmed by the developer before coding. |
| **init** | `/init` | Discover or scaffold `/context/`; creates `index.md` only when missing. Never overwrites. |
| **explore** | explore agent | Read-only repository discovery — files, patterns, dependencies, risks, context gaps. |
| **research** | research agent | Strictly read-only evidence gathering with cited facts and confidence labels. |
| **review** | `/review` | Three-layer review (plan alignment, system integrity, production readiness) with V0/R1/R2 gates and severity. |
| **quality** | on demand — debugging | Reproduce, isolate, fix, and prevent; escalate feature review to `review` and systemic audits to `audit`. |
| **audit** | `/audit` | Read-only systemic audit of security, performance, and practices; the developer owns fix decisions. |
| **remember** | `/remember save` / `/remember restore` | Secret-safe session memory; never resets or deletes dispatch state. |

### Optional

Opt-in skills are inert unless invoked; `worktrees` and `clonedeps` confirm before every relevant operation. Non-small work invokes `worktrees` before writers; Small work remains in the current checkout unless requested otherwise.

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **codemap** | `/codemap` | Explicitly requested read-only structural map; never mutates. |
| **worktrees** | `/worktrees` | Plan for safely isolating Git work; confirmation before every command. |
| **clonedeps** | `/clonedeps` | Plan for copying pinned dependency source; confirmation before every operation. |
| **simplify** | `/simplify` | Opt-in, behavior-preserving simplification proposal after a review finding or explicit request. |
| **imprint** | `/imprint` | Optional UI pattern capture. Runs only when requested; asks before creating a missing registry; no required or canonical path. |

### Frontend reference (agent library)

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **impeccable** | frontend agent | Design, redesign, critique, polish, and optimize frontend interfaces. |
| **ui-ux-pro-max** | searchable reference | UI/UX design intelligence: styles, palettes, font pairings, UX guidelines, motion presets, charts. |

## Examples

### Start a feature

```
opencode agent set orchestrator
/init                    # scaffold or discover project context (once)
> Add a paginated blog with category filtering and search
# Orchestrator: explore → architect → explicit human confirmation →
# dispatch build tasks → test → R1 review gate → complete
/remember save           # end of session
```

### Quick fix without the orchestrator

```
opencode agent set build
> Fix the broken link on the homepage hero button
# Small class: direct implementation with a V0 self-check
```

### Resume next session

```
opencode agent set orchestrator
/remember restore
```

## Credits

Built on the clean, opinionated skill design of [JSM Agent Skills](https://github.com/JavaScript-Mastery-Pro/jsm-agent-skill) and the agent-orchestration direction of [oh-my-opencode-slim](https://github.com/alvinunreal/oh-my-opencode-slim), both MIT.

## License

MIT
