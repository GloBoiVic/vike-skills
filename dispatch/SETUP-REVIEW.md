# OpenCode AI Workflow Setup — Audit Report

**Date:** 2026-08-01  
**Scope:** `~/.config/opencode/opencode.jsonc`, `vike-skills/skills/*`, `vike-skills/dispatch/*`, `vike-skills/README.md`, `~/.agents/skills/impeccable/SKILL.md`, `~/.agents/skills/ui-ux-pro-max/SKILL.md`  
**Method:** Manual review of 14 source files, full config, skill definitions, and archive remnants.

---

## Executive Verdict

**This setup is architecturally sound but has 2 critical defects, 7 important issues, and 9 minor issues that undermine reliability and documentation accuracy.**

The two critical defects — an unsupported `fallback_model` config key (silently ignored) and a Tier 3 premium review tier that cannot be realized — mean the system will not behave as intended under failure conditions and will never deliver the promised extra-thorough security reviews. The README also lists wrong models for the reviewer and backend agents, which will confuse anyone relying on the documentation.

---

## 1. Agent Roles, Modes, Visibility, and Delegation Permissions

### Verdict: Well-structured hierarchy, but gaps in permission completeness and a silent privilege blind spot.

### 1.1 Agent mode distribution

| Agent | Mode | Hidden | Correct? |
|-------|------|--------|----------|
| orchestrator | primary | (default) | ✅ |
| explore | subagent | hidden ✅ | ✅ |
| architect | subagent | hidden ✅ | ✅ |
| frontend | subagent | hidden ✅ | ✅ |
| reviewer | subagent | hidden ✅ | ✅ |
| documenter | subagent | hidden ✅ | ✅ |
| build | primary | (default) | ✅ |
| backend | subagent | hidden ✅ | ✅ |
| tester | subagent | hidden ✅ | ✅ |
| general | primary | (default) | ✅ |

Eight of ten agents are `hidden: true` subagents, which is correct — they are only invoked via the orchestrator and should not appear in the agent selector. The `build` and `general` agents are `primary` (top-level invocable), which is reasonable for independent implementation and one-off work.

### 1.2 Permission analysis

**Orchestrator** (opencode.jsonc, lines 26–39) has explicit task permission allows for 8 subagent types.  
→ **Minor issue:** `general` is missing from this allow list. If the orchestrator needs to dispatch a one-off task via the `general` agent, the Task tool will deny it. Likely fine in practice since the orchestrator uses specialized agents, but inconsistent.

**Build** (lines 101–103): `task.*: deny`, no `skill` block at all.  
→ **Acceptable.** Build is a "write code" agent that should not delegate.

**Backend** (lines 115–117): `task.*: deny`, no `skill` block.  
→ **Acceptable.** Same reasoning.

**Tester** (lines 125–127): `task.*: deny`, no `skill` block.  
→ **Acceptable.**

**Architect** (lines 52–56): Has `skill.architect: allow` but no task permissions.  
→ **Acceptable.** Architect is a read-only planning subagent; it should use the `architect` skill and not delegate.

**Frontend** (lines 64–69): Has `skill.impeccable: allow` and `skill.imprint: allow`.  
→ **Important issue:** The `impeccable` skill (v3.9.1) is a 176-line design workflow with script runners, multiple reference files, command routing rules, and user-interaction steps. It is designed for **direct, conversational design sessions**. Expecting a hidden subagent on `opencode/kimi-k2.5` to silently run this as a skill is unrealistic. The subagent lacks the environment (script paths, reference files, browser, user interaction loop) to execute it faithfully. See §5 for the override concern.

### 1.3 `general` agent blind spot

`general` (lines 105–108) has **no model, no prompt, no permissions.** It inherits whatever OpenCode's top-level default model is.  
→ **Important issue:** If no default model is set at the OpenCode application level, `general` may silently use an unexpected model or fail at invocation time. The agent is also completely unconstrained — it can do anything the base model can do with no workflow guardrails. This is acceptable for "one-off tasks" but the lack of documentation or defaults makes behavior unpredictable.

---

## 2. Model Routing and Review Tiers

### Verdict: Tier 3 premium review is architecturally unimplementable.

### 2.1 Current model assignment

| Agent | Configured Model | README Claims | Match? |
|-------|-----------------|---------------|--------|
| orchestrator | `opencode/gpt-5.6-luna` | GPT-5.6 Luna | ✅ |
| explore | `opencode/deepseek-v4-flash` | DeepSeek V4 Flash | ✅ |
| architect | `opencode/gpt-5.6-luna` | GPT-5.6 Luna | ✅ |
| build | `opencode/gpt-5.2-codex` | GPT-5.2 Codex | ✅ |
| frontend | `opencode/kimi-k2.5` | Kimi K2.5 | ✅ |
| backend | `opencode/deepseek-v4-pro` | DeepSeek V4 Flash | ❌ **WRONG** |
| reviewer | `opencode/gpt-5.6-luna` | DeepSeek V4 Flash | ❌ **WRONG** |
| tester | `opencode/deepseek-v4-flash` | DeepSeek V4 Flash | ✅ |
| documenter | `opencode/deepseek-v4-flash` | DeepSeek V4 Flash | ✅ |
| general | *(none — falls to default)* | (default model) | ❓ Undefined |

**Critical — README documentation errors** (opencode.jsonc lines 72, 111; README lines 45–46):
- **Backend:** Config uses `deepseek-v4-pro`; README claims "DeepSeek V4 Flash". These are different model tiers.
- **Reviewer:** Config uses `gpt-5.6-luna`; README claims "DeepSeek V4 Flash". This is a completely different model family that would fundamentally change review quality and cost.

### 2.2 The Tier 3 problem

The orchestrator skill (orchestrate/SKILL.md, lines 62–66) and opencode.jsonc (lines 22–25) define three review tiers:

| Tier | Model | Orchestrate/SKILL.md says | Reality |
|------|-------|--------------------------|---------|
| 1 | DeepSeek Flash | Skip formal review | ✅ Implementable (intentional skip) |
| 2 | GPT-5.6 Luna | Dispatch reviewer normally | ✅ Implementable (reviewer is hardcoded to Luna) |
| 3 | "Premium" | Dispatch with "Tier 3" flag | ❌ **Not implementable** |

**Critical — Tier 3 cannot work for three reasons:**

1. **Fixed reviewer model** (opencode.jsonc, line 72): The `reviewer` agent definition has `"model": "opencode/gpt-5.6-luna"` hardcoded. There is no mechanism in OpenCode's agent config for the orchestrator to dynamically select a different model when dispatching a subagent. The reviewer always runs on Luna, regardless of the tier flag.

2. **"Premium" is unspecified** (orchestrate/SKILL.md, line 66; opencode.jsonc, line 25): The config never defines what "Premium" means — no model name, no agent ID, no fallback. Is it Claude Opus? GPT-5.6 Pro? The orchestrator prompt says "ask three questions before using premium model" but never names which premium model to use.

3. **Tier flag is informational only** (orchestrate/SKILL.md, line 72–73): The prompt tells the reviewer to "perform an extra-thorough review" when the brief contains the "Tier 3" flag. But the flag is just text in the Task tool's prompt — it changes nothing about model selection, tool access, or review depth. It relies entirely on the agent's discretion to inspect its own prompt.

**Recommendation:** Either (a) create a separate `reviewer-premium` agent entry with a stronger model (e.g., Claude Opus or GPT-5.6 Pro) and update the orchestrator to dispatch it for Tier 3, or (b) remove the Tier 3 concept entirely and document that all reviews run on Luna with tier-appropriate prompt intensity.

---

## 3. `fallback_model` — Supported Setting or Unknown Option?

### Verdict: NOT a supported OpenCode setting. It is silently ignored.

**opencode.jsonc, line 100:**
```jsonc
"fallback_model": "opencode/deepseek-v4-pro"
```

This key appears under the `build` agent definition. **OpenCode does not support a `fallback_model` property in any documented agent schema.** The config schema (`https://opencode.ai/config.json`) has no such field.

**What happens:**
- The JSONC parser ignores unknown keys without error.
- When the `build` agent's primary model (`gpt-5.2-codex`) fails — due to rate limiting, overload, or transient API error — there is **no fallback behavior**. The task fails or retries on the same model.
- The developer likely expected graceful degradation to `deepseek-v4-pro`, but the configuration is completely inert.

**Critical — silent failure risk:** Under production load, if `gpt-5.2-codex` is unavailable, the build agent will fail with no automatic fallback. The `fallback_model` key creates an illusion of resilience that does not exist.

**Recommendation:** Remove the `fallback_model` key (it is noise). If fallback behavior is genuinely needed, implement it at the application level (OpenCode's own retry/fallback config) or write it into the `build` agent's prompt as instructions ("If the primary model fails, retry with a fallback model" — though this is also unreliable since the agent can't change its own model).

---

## 4. Context and Dispatch Workflows, Source-of-Truth Consistency, Token Efficiency

### Verdict: Well-designed but verbose; token efficiency could be improved.

### 4.1 Source-of-truth architecture

The setup establishes a clean three-layer model:
- **`context/`** — durable project knowledge (created by `init`, authoritative once filled in)
- **`dispatch/`** — ephemeral session history and task tracking (created per feature)
- **`memory.md`** — cross-session state bridge (created by `remember`)

This is well-considered. The `remember` skill correctly references `dispatch/` and `context/` as the factual record rather than storing duplicate data in `memory.md`.

### 4.2 Issues

**Important — `/dispatch/` structure defined in two places with slight differences:**

| File | `orchestrate/SKILL.md` (line 73) | `dispatch/SKILL.md` (lines 102–112) |
|------|----------------------------------|--------------------------------------|
| PLAN.md | ✅ | ✅ |
| ARCHITECTURE.md | ✅ | ✅ |
| TASKS.md | ✅ | ✅ |
| DECISIONS.md | ✅ | ✅ |
| REVIEW.md | ✅ | ✅ |
| MODEL-LOG.md | ✅ | ✅ |
| EXPLORATION.md | ✅ | ✅ |
| COMPLETED.md | ✅ | ✅ |

Both match. No conflict here, but see §5 on whether `quality/SKILL.md` writes to `REVIEW.md` while `review/SKILL.md` also expects to write there.

**Minor — explore agent duplicates orchestrator's doc-reading duties:**
`explore/SKILL.md` line 20 says "Check remaining docs (`/dispatch/*` and any project docs not covered by the index)". The orchestrator already reads `/dispatch/` in its Intake step (orchestrate/SKILL.md line 23). Having the explore agent re-read these wastes tokens. Recommendation: `explore` should focus on codebase patterns and let the orchestrator provide dispatch context via the task brief.

### 4.3 Token efficiency concerns

**All skill files are long.** While this is a minor concern at the SKILL.md level (loaded once per session), the cumulative cost adds up:

| Skill | Lines | When loaded |
|-------|-------|-------------|
| `quality` | 237 | Every review (reviewer agent) |
| `audit` | 273 | On-demand (expensive) |
| `init` | 241 | First session or when invoked |
| `imprecable` | 176 | Frontend tasks |
| `imprint` | 256 | UI component tasks |

**Quality skill is overkill for routine reviews** (quality/SKILL.md has 5 sections; Section C — Feature Review — is the only relevant one for the reviewer agent). The reviewer's prompt says "Use the quality skill for structured review criteria" (opencode.jsonc line 76), which loads the entire 237-line monster file. Every review pays for debugging, recovery, security, and performance sections it doesn't use.

**Recommendation:** If OpenCode supports sectioned skill loading (e.g., `@quality/review`), use it. Otherwise consider splitting `quality` into smaller, composable skills or at minimum adding an instruction at the top of `quality/SKILL.md` telling the agent to read only the section it needs and `break` after reading it — which is already partially done (line 6: "Use the one that matches the current situation").

---

## 5. Duplicate/Overlapping Skills and Confusing Instructions

### Verdict: Three significant overlap zones exist.

### 5.1 `quality` vs. `review` vs. `audit` — triple overlap

| Skill | Sections | Overlap with |
|-------|----------|-------------|
| `review` | 3-layer feature review | `quality` Section C (identical) |
| `quality` | Debugging + Recovery + Review + Security Audit + Performance Audit | `review` (Section C), `audit` (Sections D+E) |
| `audit` | Security + Performance + Best Practices | `quality` Sections D+E |

**Important — `quality` Section C (lines 123–159) is a verbatim copy of `review/SKILL.md`'s three-layer review.** The structure, language, and severity guide are identical. This means:
- If one is updated, the other will drift silently.
- The reviewer agent is told to use `quality` (which includes Section C), but the dispatch skill tells task reviewers to use `review` (dispatch/SKILL.md line 88). Which wins?

**Important — `quality` Sections D+E (Security + Performance audit, lines 163–226) overlap with `audit/SKILL.md` (273 lines).** The `audit` skill is more thorough (adds Best Practices and false-positive discipline), but the core checks are the same. Running both would be wasteful; relying on just one misses the other's unique content.

### 5.2 Archived skills remain on disk

`archive/debug/SKILL.md` (197 lines) and `archive/recover/SKILL.md` (251 lines) are retired but still present. These are **excluded from OpenCode's skill paths** (opencode.jsonc lines 10–12 points only to `vike-skills/skills` and `.agents/skills`), so they won't be loaded — but they create confusion for anyone reading the repository.

**Minor issue:** The `quality` skill's Sections A and B (Debugging and Recovery) are direct adaptations of these archive skills. The prompts differ slightly, so there is no active inconsistency, but the archive should be cleaned up to avoid future confusion.

### 5.3 `impeccable` override concern

`frontend` agent (opencode.jsonc line 63) says: "Implement UI components using the project's design system and existing patterns, and **follow impeccable design standards**."

This instructs a Kimi K2.5 subagent to follow the `impeccable` skill — a 176-line, script-driven design workflow with 21 sub-commands, brand register references, and a complex setup procedure. The `impeccable` skill is activated via the `skill` permission system, but its execution model (run a context script, read reference files, ask user questions) is incompatible with a hidden subagent that has no user interaction channel.

**Important — this instruction will either be silently ignored or produce degraded output.**

---

## 6. General One-Off Agent Behavior and Default Model Behavior

### Verdict: `general` agent is a safety gap.

The `general` agent (opencode.jsonc, lines 105–108) is defined as:
```jsonc
"general": {
  "mode": "primary",
  "description": "General-purpose agent for one-off tasks outside the orchestrator workflow"
}
```

**Important issues:**
1. **No `model`** — falls through to OpenCode's application-level default, which is unspecified in this config. If OpenCode itself has no default, the agent may fail or use an unpredictable model.
2. **No `prompt`** — the agent has zero behavioral guardrails. It will behave exactly like the raw model with no workflow, no constraints, no skill references.
3. **No `permission`** — the agent has unrestricted access to all tools and skills.

For a "one-off task" agent this may be intentional (maximum freedom), but it means **`general` is the most powerful agent in the system while being the least configured.** If the user invokes `general` by accident or via the orchestrator (which can't, per §1.2), there are no safety rails.

**Recommendation:** Either give `general` a clear default model and a minimal safety prompt, or add a comment explaining it's intentionally unconfigured for flexibility.

---

## 7. README Accuracy and Missing Usage Examples

### Verdict: Three model assignments wrong; no CLI invocation examples.

### 7.1 Model table errors

| Agent | Actual (opencode.jsonc) | README Claims | Severity |
|-------|----------------------|---------------|----------|
| backend | `deepseek-v4-pro` (line 111) | "DeepSeek V4 Flash" (line 45) | Critical |
| reviewer | `gpt-5.6-luna` (line 72) | "DeepSeek V4 Flash" (line 46) | Critical |
| explore | `deepseek-v4-flash` (line 43) | "DeepSeek V4 Flash" (line 42) | ✅ Correct |

These are not ambiguous naming issues — `deepseek-v4-pro` vs `deepseek-v4-flash` are different model tiers with different capabilities and costs. The reader building expectations around the reviewer agent will be misled about review quality.

### 7.2 Missing usage examples

The README describes the workflow architecture and `/dispatch/` structure but provides **no concrete CLI examples** showing how to invoke the system:

- No `opencode --agent orchestrator` or equivalent
- No example of running `/init` or `/dispatch` in a session
- No example of `/remember save` / `/remember restore` in practice
- No example of the "Engineering Loop" as executable commands

Compare with the rich command tables and invocation formats in skills like `init`, `audit`, `imprint`, and `remember` — the README is significantly less usable than the individual skill files suggest.

**Recommendation:** Add a "Quick Start" section with:
```bash
# Start a session with the orchestrator
opencode --agent orchestrator "Build a login page"

# Or in-session: set up a new project
# /init
# (project type detected, context scaffolded)

# Or: end a session
# /remember save
```

### 7.3 Other README inaccuracies

- Line 9 says "Ten skills." There are 10 SKILL.md files in `vike-skills/skills/`, which is correct, but `archive/` contains 5 more, and `.agents/skills/` has `impeccable` and `ui-ux-pro-max`. The count "ten" only refers to the first-party skills. This could confuse users who see more in their agent's skill list.

---

## 8. Minor Issues

| # | File | Line(s) | Issue | Recommendation |
|---|------|---------|-------|---------------|
| 1 | `dispatch/SKILL.md` | 2 | YAML `name: dispatch` but description references `explore`, `architect`, `build`, etc. that are agents, not skills | No fix needed — the skill instructs agent delegation |
| 2 | `orchestrate/SKILL.md` | 17 | "Establish project context — the orchestrator owns context initialization" — but the orchestrator prompt in opencode.jsonc (line 20) says "never write application code" — this is about context, not app code; consistent ✅ | — |
| 3 | `quality/SKILL.md` | 6 | "This skill replaces the need for separate debug, recover, review, and audit skills" — this is overstated, especially since `review` and `audit` still exist as separate skills with unique content | Either merge them or remove this claim |
| 4 | `explore/SKILL.md` | 20 | "Check remaining docs (`/dispatch/*` and any project docs not covered by the index)" — duplicates orchestrator | Remove this line; let orchestrator own dispatch context |
| 5 | `opencode.jsonc` | 100 | `fallback_model` is unsupported | Remove key; add retry logic at app level if needed |
| 6 | `opencode.jsonc` | 105–108 | `general` agent has no model, prompt, or permissions | Add a comment or minimal config |
| 7 | `dispatch/DECISIONS.md` | 3–6 | Date `2026-08-01` matches today; the decisions file only has one entry | This is fine for a young project |
| 8 | `dispatch/COMPLETED.md` | — | Not checked, but expected to exist per the /dispatch/ template | Confirm it exists |
| 9 | `README.md` | 38–49 | Agent table uses code font for models but no consistent naming convention (mixed "DeepSeek V4 Flash" and "GPT-5.6 Luna") | Standardize model names to match opencode.jsonc format |

---

## Findings Summary

| Severity | Count | Key Areas |
|----------|-------|-----------|
| **Critical** | 2 | `fallback_model` silently ignored (opencode.jsonc:100); Tier 3 premium review unimplementable (orchestrate/SKILL.md:62–66) |
| **Important** | 7 | README model table wrong for reviewer & backend; `general` agent underconfigured; `quality` overlaps with `review`/`audit`; `frontend` can't meaningfully use `impeccable`; `quality` loaded in full for every review; `orchestrator` can't dispatch `general`; explore duplicates orchestrator's dispatch read |
| **Minor** | 9 | Various file-level inconsistencies, missing usage examples, archive cleanup, token efficiency notes |

---

## Recommendations (Prioritized)

1. **[Critical]** Remove `fallback_model` from opencode.jsonc:100 — it does nothing.
2. **[Critical]** Either create a `reviewer-premium` agent with a stronger model or delete Tier 3 from orchestrator documentation.
3. **[Important]** Fix README model table: backend → `deepseek-v4-pro`, reviewer → `gpt-5.6-luna`.
4. **[Important]** Add a model and minimal prompt to the `general` agent.
5. **[Important]** Either merge `quality`/`review`/`audit` into one unified skill or define which skill the reviewer should actually load (and update dispatch/SKILL.md line 88 to match).
6. **[Important]** Add usage examples to README (Quick Start section).
7. **[Minor]** Remove `/dispatch/*` from explore agent's workflow (orchestrator already handles it).
8. **[Minor]** Clean up `archive/` directory or add a README there explaining the dead skills.
9. **[Minor]** Consider splitting `quality` into smaller composable skills or adding better inline guidance to skip unused sections.
