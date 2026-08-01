# OpenCode Setup Review

## Scope

Audit the global OpenCode configuration, agent roles, model routing, permissions, skills, dispatch/context workflows, and README usage guidance.

## Review questions

1. Does the agent hierarchy match the intended workflow?
2. Are model routes and review tiers implementable with the current OpenCode configuration?
3. Are permissions safe and sufficient for delegation?
4. Are `/dispatch/` and `/context/` durable, discoverable, and token-efficient?
5. What issues or maintenance risks remain?

## Deliverables

- Independent review report in `dispatch/SETUP-REVIEW.md`.
- Updated `README.md` with practical workflow examples.
