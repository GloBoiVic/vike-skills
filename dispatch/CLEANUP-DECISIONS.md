# Cleanup Decisions

## 2026-08-01 — Explicit premium reviewer

Tier 3 requires a separate `reviewer-premium` agent rather than an informational prompt flag. This makes the route observable and manually configurable.

## 2026-08-01 — Canonical quality responsibilities

`review` owns feature review, `audit` owns systemic security/performance/practice audits, and `quality` owns debugging/recovery. This avoids loading duplicate checklists for every review.

## 2026-08-01 — No custom fallback

Remove unsupported `fallback_model`. OpenCode's own model availability/default behavior remains authoritative.
