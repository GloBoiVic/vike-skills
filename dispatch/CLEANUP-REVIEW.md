# Cleanup Review — Workflow Cleanup (Phase 2, Task 4)

**Date:** 2026-08-01  
**Reviewer:** code-reviewer (opencode/deepseek-v4-flash)  
**Scope:** `opencode.jsonc` · `skills/orchestrate/SKILL.md` · `skills/quality/SKILL.md` · `skills/review/SKILL.md` · `skills/audit/SKILL.md` · `skills/explore/SKILL.md` · `README.md`  
**Plan:** `dispatch/CLEANUP-PLAN.md`  
**Decisions:** `dispatch/CLEANUP-DECISIONS.md`  
**Implementation report:** `dispatch/CLEANUP-IMPLEMENTATION-REPORT.md`

---

## Layer 1 — Plan Alignment

| # | Plan Checkpoint | Status | Evidence |
|---|-----------------|--------|----------|
| 1 | No `fallback_model` remains | **PASS** | No `fallback_model` key exists in `opencode.jsonc`. The only references are in `README.md` line 78 (documenting its absence) and in cleanup dispatch files (history, not config). Grep across `~/.config/opencode/` returns zero matches. |
| 2 | `reviewer-premium` exists, hidden, configurable, dispatchable | **PASS** | Present at `opencode.jsonc` lines 84–95. `hidden: true`. Model `opencode/gpt-5.6-sol` — prompt explicitly notes "manually swappable". Orchestrator task permissions include `reviewer-premium: allow` (line 36). `orchestrate/SKILL.md` lines 67, 74 dispatch `reviewer-premium` for Tier 3. |
| 3 | `reviewer` loads review; `reviewer-premium` loads audit | **PASS** | `reviewer`: `"skill": { "review": "allow" }` (line 80). `reviewer-premium`: `"skill": { "audit": "allow" }` (line 92). |
| 4 | Tier 1/2/3 routing explicit and consistent | **ISSUES FOUND** | `orchestrate/SKILL.md` lines 61–76: explicit, correct routing. However, the orchestrator's in-config prompt (`opencode.jsonc` line 25) says _"dispatch reviewer with Tier 3 flag"_ instead of _"dispatch `reviewer-premium`"_ — see **Important-1**. |
| 5 | `quality` contains only debugging/recovery | **PASS** | `quality/SKILL.md` is 90 lines, Sections C/D/E removed. Frontmatter: _"Systematic debugging and failure recovery. Diagnose the cause before fixing; hand feature reviews to review and systemic audits to audit."_ Top-level heading: _Quality — Debugging and Recovery_. Explicit handoff lines 8, 16, 88–89. |
| 6 | `explore` no longer rereads all dispatch files | **PASS** | `explore/SKILL.md` line 20: "do not reread all /dispatch/* files". Orchestrator exploration step (orchestrate line 29) reads only `/dispatch/EXPLORATION.md`, consistent. |
| 7 | `general` has no model override + minimal safety prompt | **PASS** | No `model` field on `general` agent (lines 117–121). Prompt includes: _"Read AGENTS.md and relevant context/index.md when available, inspect project conventions before changing files, and avoid destructive actions unless explicitly requested."_ |
| 8 | README matches actual config | **PASS** | Agent table models match `opencode.jsonc` exactly. Review tier descriptions match `orchestrate/SKILL.md`. Quality skill description: _"Debugging and failure recovery. Use `review` for feature reviews and `audit` for systemic audits."_ Audit skill description matches. Workflow examples are usable and reference correct agents. |
| 9 | JSONC/frontmatter/diff consistency | **ISSUES FOUND** | See **Important-2** and **Minor-1** below. |

---

## Layer 2 — System Integrity

### Checked

- **Skill boundaries respected:** `review` owns feature review, `audit` owns systemic audit, `quality` owns debugging/recovery. These boundaries are clean in the config (skill permissions) and in the skill files.
- **Architecture boundaries:** orchestrator never writes code (lines 98–102 of its skill). Quality hands off to review/audit rather than duplicating them.
- **Frontmatter integrity:** All 10 skill files in `skills/` have valid `name`/`description` frontmatter.
- **JSONC validity:** `opencode.jsonc` parses cleanly (state-machine JSONC safe).
- **No silent drift:** The plan explicitly forbids modifying application source code — no evidence of that.
- **impeccable / imprint preserved:** Both remain in `frontend` agent permissions and prompt.

### Issues Found

**Important-1 — Orchestrator prompt in opencode.jsonc dispatches wrong agent for Tier 3**

The orchestrator's in-config system prompt (line 25) states:

> Tier 3 (premium): authentication, security, payments, major system redesigns — dispatch **reviewer** with Tier 3 flag for extra-thorough review.

But `orchestrate/SKILL.md` (lines 67, 74) and the cleanup plan (item 3) specify:

> Tier 3 → dispatch **`reviewer-premium`** with brief + files + constraints + **"Tier 3: security-critical review required"** flag.

**Impact:** The working orchestrator prompt tells the LLM to dispatch the regular `reviewer` agent with a flag, not the `reviewer-premium` agent. While the SKILL.md is read when the `orchestrate` skill loads, the system prompt is loaded first and would override the skill instructions. A literal-reading LLM would dispatch `reviewer` (Luna) for security-critical work instead of `reviewer-premium` (Sol), defeating the entire purpose of Tier 3.

**Fix:** Change `opencode.jsonc` line 25 from:
```
— dispatch reviewer with Tier 3 flag for extra-thorough review.
```
to:
```
— dispatch reviewer-premium with a "Tier 3: security-critical review required" flag.
```

**Important-2 — Reviewer prompt references `quality` skill without permission**

The `reviewer` agent's prompt (line 77) says:

> Use the **quality** skill for structured review criteria.

But the `reviewer` agent's `permission.skill` block (lines 79–82) only allows `review: allow`. It does **not** have `quality: allow`.

**Impact:** At runtime, when the reviewer agent attempts to load the `quality` skill per its prompt instructions, OpenCode will deny the request (skill not permitted). The agent will either fail silently, produce a confusing error, or skip the instruction entirely — none of which match the intended behavior.

**Fix:** Change the prompt from "Use the quality skill for structured review criteria" to "Use the review skill for structured review criteria" (or simply remove the sentence, since the `reviewer` agent has permission for `review` which contains its own three-layer criteria).

**Minor-1 — Reviewer description still says "quality verification"**

Line 76: `"description": "Code review and quality verification"`

The `reviewer` agent is now scoped to `review` skill only. The description implies it also handles "quality verification" which is now the domain of `audit` and `reviewer-premium`. Trivially misleading.

**Fix:** Change to `"Code review and feature verification"` or `"Feature review"`.

---

## Layer 3 — Production Readiness

| Check | Result |
|-------|--------|
| **Error handling** — missing skill permission will cause runtime failures | **❌ Important-2**: Reviewer cannot load `quality` skill it's told to use |
| **Edge cases** — orchestrator may dispatch wrong agent for Tier 3 | **❌ Important-1**: Prompt vs. SKILL.md mismatch |
| **False security** — Tier 3 appears configured but prompt routes to wrong agent | **❌ Important-1**: Creates illusion of premium review without delivering it |
| **Config correctness** — all JSONC keys valid, no deprecated keys | **✅** No `fallback_model`, no unknown properties |
| **README usability** — examples match actual workflow | **✅** Examples are accurate |

---

## Summary

**Critical:** 0  
**Important:** 2  
**Minor:** 1  

### Verdicts

| Check | Verdict |
|-------|---------|
| 1. No `fallback_model` | ✅ PASS |
| 2. `reviewer-premium` exists + hidden + configurable + dispatchable | ✅ PASS |
| 3. `reviewer` loads review; `reviewer-premium` loads audit | ✅ PASS |
| 4. Tier 1/2/3 routing explicit and consistent | ⚠️ PASS WITH ISSUES |
| 5. `quality` only debugging/recovery | ✅ PASS |
| 6. `explore` no dispatch re-read | ✅ PASS |
| 7. `general` no model + safety prompt | ✅ PASS |
| 8. README matches config | ✅ PASS |
| 9. JSONC/frontmatter/diff consistency | ⚠️ PASS WITH ISSUES |

**2 Important issues found.** Both are in `opencode.jsonc` and both affect runtime behavior: the orchestrator prompt dispatches the wrong agent for Tier 3, and the reviewer agent references a skill it has no permission to use. These should be fixed before the cleanup is considered complete.

---

## Detailed Findings

### Important-1 — Orchestrator prompt routes Tier 3 to `reviewer` instead of `reviewer-premium`

- **File:** `/Users/vike/.config/opencode/opencode.jsonc` line 25
- **What:** `dispatch reviewer with Tier 3 flag for extra-thorough review.`
- **Should be:** `dispatch reviewer-premium with a "Tier 3: security-critical review required" flag.`
- **Evidence:** Plan item 3: "Route Tier 3 work to `reviewer-premium`; Tier 2 to `reviewer`." Decision 2026-08-01: "Tier 3 requires a separate `reviewer-premium` agent." `orchestrate/SKILL.md` line 67: "Dispatch `reviewer-premium`."
- **Severity: Important** — If the orchestrator follows its system prompt, Tier 3 reviews will run on the regular `reviewer` (Luna) instead of `reviewer-premium` (Sol), undermining the security-review guarantee.

### Important-2 — Reviewer agent prompt references `quality` skill without permission

- **File:** `/Users/vike/.config/opencode/opencode.jsonc` line 77
- **What:** `Use the quality skill for structured review criteria.`
- **Problem:** `reviewer`'s `permission.skill` only allows `review: allow`. The `quality` skill is not permitted. The agent will fail when trying to load it.
- **Severity: Important** — Runtime failure mode. The reviewer will not have access to the criteria it's instructed to use.

### Minor-1 — Reviewer description includes "quality verification"

- **File:** `/Users/vike/.config/opencode/opencode.jsonc` line 76
- **What:** `"description": "Code review and quality verification"`
- **Suggestion:** `"description": "Code review and feature verification"` — since "quality verification" (systemic auditing) is now the domain of `audit` and `reviewer-premium`.
- **Severity: Minor** — Cosmetic; doesn't affect behavior.

---

## Omissions (not in plan scope, noted for awareness)

- The `SETUP-REVIEW.md` and `SETUP-REVIEW-PLAN.md` files remain in `/dispatch/` but document the *pre-cleanup* state. They are historical records and do not affect the current configuration.
- The orchestrator prompt in `opencode.jsonc` (lines 19–25) still uses model names ("DeepSeek Flash", "GPT-5.6 Luna") alongside agent names. This is not incorrect but introduces a second naming convention (model-based vs. agent-based) that could be unified.
