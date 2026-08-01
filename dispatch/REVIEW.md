# Review — Phase 2: Adaptive Project Context

Date: 2026-08-01
Reviewer: reviewer (opencode/deepseek-v4-flash-free)
Reviewed: `skills/init/SKILL.md`, `skills/orchestrate/SKILL.md`, `skills/explore/SKILL.md`, `skills/remember/SKILL.md`, `~/.config/opencode/opencode.jsonc`, `dispatch/PLAN.md`, `dispatch/ARCHITECTURE.md`, `dispatch/phase2-implementation-report.md`

## Layer 1 — Plan Alignment: PASS

All three PLAN tasks covered. Task 1 (init skill): created at `skills/init/SKILL.md` with detect → inventory/create branching, project-type inference, core + optional templates, no-overwrite guarantees. Task 2 (workflow integration): orchestrate, explore, remember, and frontend guidance all read `AGENTS.md` + `context/index.md` first and load selectively. Task 3 (tracking): TASKS.md/MODEL-LOG.md/COMPLETED.md/report updated; review was the remaining open item and is closed here.

Constraints from PLAN.md verified:
- Never overwrite/delete existing context files — enforced in init (core principles, inventory step 4), orchestrate (Prohibited Actions), ARCHITECTURE.md.
- Index created only when missing; missing recommended docs reported, not scaffolded — init Inventory mode step 3.
- Core docs (`project-brief.md`, `tech-stack.md`, `architecture.md`, `coding-standards.md`) always generated; optional docs adaptive by project type (web/backend/library/full-stack mapping) — init Recommended Docs.
- `impeccable` not split — confirmed intact (`~/.agents/skills/impeccable`, v3.9.1); perms unchanged.

## Layer 2 — System Integrity: PASS

- **Criterion 1 — init distinguishes existing vs. absent context/.** PASS. Step 2a (Inventory mode) vs. Step 2b (Create mode) are distinct and complete; project type inferred from codebase, asked only when ambiguous.
- **Criterion 2 — no overwrite/delete; index rules explicit.** PASS. "Existing files are authoritative and immutable", "Only missing things are created", "never rewrite [the index] wholesale". Index creation (missing only) and gap identification are explicit. Update rule for an existing-but-incomplete index is present ("If a doc exists but has no entry, add an entry") but sits in the Index Format section rather than the Inventory workflow — see Minor-1.
- **Criterion 3 — adaptive by project type.** PASS. Core always; optional table with "Include when" conditions plus project-type mapping.
- **Criterion 4 — agents read AGENTS.md + context/index.md first, load selectively.** PASS. Orchestrate (Intake step 1–4), explore (step 2), remember (save: read index before writing; restore: read index + AGENTS.md), and the frontend agent prompt all follow the ordering and reference index one-liners for selective loading. Bulk-reads of `context/` are explicitly discouraged.
- **Criterion 5 — orchestrator non-coding, no direct project-doc edits.** PASS. Prohibited Actions forbid application code, direct project-doc modification (delegate to documenter), and context-doc edits (init owns them; only missing files may be created). Orchestrator writes only to `/dispatch/`.
- **Criterion 6 — frontend guidance keeps impeccable/imprint, no split.** PASS. Config retains `impeccable: allow` and `imprint: allow`; prompt references both; impeccable skill untouched.
- **Criterion 7 — config valid JSONC; init discoverable.** PASS. `opencode.jsonc` validated with a state-machine JSONC stripper (comment/trailing-comma safe, including `https://` URLs) — parses clean. `skills.paths` still includes `/Users/vike/Desktop/vike-skills/skills`; `skills/init/SKILL.md` present with valid `name`/`description` frontmatter (all 10 SKILL.md files pass frontmatter check). Orchestrator model, task permissions, and MCP config unchanged.
- **Criterion 8 — no unnecessary token-heavy duplication.** PASS with minor notes. The init index template re-lists the Core/Optional docs from the Recommended Docs table (defensible: definition vs. format example). The frontend prompt restates AGENTS.md/index reading guidance for subagent self-containment (necessary; the subagent does not load the orchestrate skill). No verbatim duplication of large blocks across the changed files.

## Layer 3 — Production Readiness: PASS

- No `context/` directories were created anywhere by the implementation session (verified via `find`), confirming init was not exercised destructively and the no-overwrite behavior holds.
- State-machine validation of `opencode.jsonc` passes; frontmatter of all 10 skill files passes.
- Edge cases handled: existing index but undocumented files (add-entry rule), missing recommended docs (reported, not scaffolded), ambiguous project type (ask developer), stale index entries (left untouched — safe default; see Minor-2).
- No security surface introduced: init/explore/remember are documentation workflows; remember retains its security boundary (no secrets in memory.md).

## Verdicts

- **Spec compliance: PASS** — all PLAN/ARCHITECTURE requirements met, scope maintained (one minor scope note below), constraints honored.
- **Quality: PASS** — clear branching, explicit safety rules, consistent cross-references, no critical or important defects.

## Findings

### Critical
- None.

### Important
- None.

### Minor
1. **Index update rule not referenced from Inventory workflow.** `skills/init/SKILL.md` line 112 states the "add a missing entry / never rewrite wholesale" rule, but step 2a ("Create the index if missing") does not point to it. Orchestrate explicitly allows invoking init for an "inventory refresh"; making the update rule part of step 2a would remove ambiguity about what init does when the index exists but is incomplete.
2. **Stale index entries are never cleaned.** If a developer deletes a context doc after the index is written, init leaves the entry (correctly — init never deletes). Agents may then attempt to load a removed file. Consider a note that agents should tolerate missing docs (e.g., "if a doc is absent, skip it and flag the stale index entry").
3. **Report inaccuracy: "all 11 skills directories".** `phase2-implementation-report.md` says frontmatter was checked across "all 11 skills directories" but lists 10 names, and exactly 10 SKILL.md files exist under `skills/` (architect, audit, dispatch, explore, imprint, init, orchestrate, quality, remember, review). The count is off by one; the validation itself is fine.
4. **Report wording on pre-existing changes.** The concern about `skills/dispatch`, `skills/debug`, `skills/recover` says the working-tree changes are "consistent with an earlier optimization commit." The last commit (1cab479, 2026-07-24) did optimize `dispatch`/`debug`/`recover`, but the current deletions of `debug`/`recover` (archived to `archive/`) and the extra `dispatch` edits are **uncommitted** working-tree changes predating Phase 2 — accurate that Phase 2 did not touch them, but they are not from a commit. Worth committing or documenting separately.
5. **Orchestrator wording tension.** `skills/orchestrate/SKILL.md` says the orchestrator "owns context initialization" and then "Never initialize context yourself." Intent is clear (owns the decision, delegates the action), but the phrasing invites confusion; tighten to "owns the initialization decision; the init skill performs it."
6. **Scope note on remember.** The save-flow overhaul in `skills/remember/SKILL.md` (merge-over-overwrite, "What to preserve during updates", "Eureka moments", "Preserved from previous session") goes beyond PLAN task 2's stated scope ("use AGENTS.md plus context/index.md, then selectively load"). The changes are coherent improvements aligned with the anti-duplication goal, but they should be acknowledged in PLAN/DECISIONS as part of the change.

## Files updated in this review
- `dispatch/REVIEW.md` — this report.
- `dispatch/TASKS.md` — task 3 marked done with review verdict.
- `dispatch/MODEL-LOG.md` — review run logged.
- `dispatch/COMPLETED.md` — Phase 2 review completion appended.
