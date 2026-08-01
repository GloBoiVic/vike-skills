# OpenCode Workflow Cleanup Plan

## Goal

Make the OpenCode setup reliable and simpler to operate without fake fallback behavior, ambiguous premium review routing, or overlapping quality skills.

## Changes

1. Remove unsupported `fallback_model` configuration.
2. Add explicit hidden `reviewer-premium` agent with a manually swappable premium model.
3. Route Tier 3 work to `reviewer-premium`; Tier 2 to `reviewer`; Tier 1 skips formal review.
4. Make `review` the canonical feature-review skill and `audit` the canonical systemic security/performance skill.
5. Reduce `quality` to debugging and recovery only.
6. Add minimal safety/context guidance to the model-agnostic `general` agent.
7. Clarify frontend static-context usage and reduce Explorer dispatch-context duplication.
8. Update README and dispatch records to match the final architecture.

## Constraints

- Preserve the general agent's omitted model field.
- Do not modify application source code.
- Keep `impeccable` and `imprint` active.
- Do not claim automatic model fallback where OpenCode does not provide it.
