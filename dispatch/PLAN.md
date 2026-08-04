# Plan — Vike Skills Workflow Enforcement: Mandatory Architect + Config-Authoritative Models

## What we are building

A targeted workflow enforcement change across the Vike Skills codebase. We are tightening the orchestrator pipeline so that **every Feature, Architecture, and Security-sensitive task** requires an explicit Explore → Architect → human confirmation cycle before any implementation begins. Small tasks (typo, button change, comment fix) remain eligible to skip Architect. Separately, we are correcting the README agent-model table to reflect the actual, authoritative model assignments in `opencode.jsonc` — the README must not invent or contradict the config. The global orchestrator prompt in `opencode.jsonc` and the `orchestrate/SKILL.md` are the primary files to update; the README is corrected afterward. No changes to default_agent or to any subagent skill files beyond orchestrate.

## Tasks

### Phase A — Orchestrator & Config Enforcement

1. **Update orchestrator prompt in `opencode.jsonc`** — Rewrite the orchestrator's global system prompt to enforce: mandatory Explore → Architect → explicit human confirmation before dispatching any implementation agent for Feature/Architecture/Security-sensitive tasks. Small tasks may skip Architect. Architect produces the design blueprint; implementation agents must follow it without deviation. The prompt must explicitly prohibit skipping Architect for Feature/Architecture/Security tasks. → **orchestrator** (this is a config-level change, applied directly to the orchestrator agent definition)

2. **Update `skills/orchestrate/SKILL.md`** — Revise the complexity-classification section (lines 13–16) so that the Feature flow reads "Explorer → Architect → Build → Test → Review" (making Architect mandatory, not optional), and Security-sensitive flow likewise mandates Architect. Update the "Plan" section (line 43–49) to state that Architect produces the design blueprint and implementation agents must follow it. Ensure the "Delegate" section (line 59–66) instructs the orchestrator to refer implementation agents to the Architect's blueprint. No changes to subagent skill files. → **orchestrator**

### Phase B — Documentation Sync

3. **Update README agent-model table** — Correct the "Agent Specialization" table (lines 64–76) to match the actual model assignments in `opencode.jsonc`:
   - **build**: `opencode/gpt-5.6-luna` (was incorrectly listed as `opencode/deepseek-v4-pro`)
   - **frontend**: `opencode/gpt-5.6-luna` (was incorrectly listed as `opencode/kimi-k2.7-code`)
   - **backend**: `opencode/gpt-5.6-luna` (was incorrectly listed as `opencode/deepseek-v4-pro`)
   - **reviewer**: `opencode/deepseek-v4-flash` (was incorrectly listed as `opencode/gpt-5.6-luna`)
   - **reviewer-premium**: `opencode/deepseek-v4-pro` (was incorrectly listed as `opencode/gpt-5.6-luna (default; swap...)`)

   Also update the Review Tiers section (lines 91–93) and Complexity Routing table (lines 82–88) where they reference these models. Do not change any other README content. → **documenter**

4. **Sync README complexity routing to mandatory Architect** — Update the Complexity Routing table (lines 82–88) so the Feature row reads "Explore → Architect → Build → Test → Review (Tier 2)" and the Security-sensitive row reads "Explore → Architect → Build → Test → reviewer-premium (Tier 3)", matching the new mandatory Architect rule. → **documenter**

### Phase C — Review & Gate

5. **Implementation review** — Verify all four changes against the spec:
   - opencode.jsonc orchestrator prompt enforces mandatory Explore → Architect → confirmation for Feature/Architecture/Security
   - orchestrator/SKILL.md complexity routing makes Architect mandatory for Feature and Security-sensitive
   - orchestrator/SKILL.md states Architect blueprint is authoritative for implementation agents
   - README model table exactly matches opencode.jsonc agent model assignments
   - No changes to default_agent, no changes to subagent skill files (explore, architect, build, etc.)
   - No changes to opencode.jsonc beyond the orchestrator prompt
   → **reviewer**

6. **Documentation sync** — After the review passes (or after any fix loop), ensure all dispatch files are consistent, update MODEL-LOG.md, and confirm the final state. → **documenter**

## Complexity tier

Architecture (workflow definition affects all future Vike Skills sessions, mandates how orchestrator dispatches work, and touches config + skill files + README).

## Relevant context

- **Config is authoritative:** `/Users/vike/.config/opencode/opencode.jsonc` defines actual agent models. README must match it exactly — the README currently lists `build: deepseek-v4-pro`, `frontend: kimi-k2.7-code`, `backend: deepseek-v4-pro`, `reviewer: gpt-5.6-luna`, and `reviewer-premium: gpt-5.6-luna` which all contradict the config.
- **Mandatory Architect rule:** Feature (new page, API, component), Architecture (multi-tenancy, system redesign), and Security-sensitive (auth, payments) tasks must ALL pass through Explore → Architect → human confirmation. Small tasks (typo, button, comment) may skip Architect.
- **Architect owns the blueprint:** Once the architect produces the implementation plan and the human confirms it, implementation agents must follow that blueprint without deviation. This prevents implementation drift.
- **No default_agent changes:** The instruction explicitly says "possibly default_agent only if explicitly approved" — we do not assume approval and will not touch it.
- **Files in scope:** `~/.config/opencode/opencode.jsonc` (orchestrator prompt), `skills/orchestrate/SKILL.md`, `README.md`.
- **Files out of scope:** All subagent skill files (architect, build, explore, frontend, backend, reviewer, tester, documenter, etc.), all non-README docs, all source code, all tests.
- **Sequencing:** Phase A (config + skill) → Phase B (README sync) → Phase C (review + gate). Each task must complete before the next begins — no parallel implementation to avoid merge conflicts.
