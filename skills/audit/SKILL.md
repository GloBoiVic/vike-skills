---
name: audit
description: Read-only systemic audit for security, performance, and engineering-practice risks; returns an evidence-based severity report for the developer to triage.
---

# Audit

Use for codebase-wide or systemic risk, before a major release or after a substantial development cycle. Use `review` for verification of one feature against its plan. This skill identifies and reports; it never fixes.

## Contract

1. Define the requested scope (security, performance, practices, a path, or all three).
2. Inspect targeted evidence rather than reading every file: manifests and lockfiles for dependencies; routes/middleware for auth; relevant configuration for headers/build; focused searches and surrounding code for injection, secrets, queries, rendering, and resource lifetime.
3. Validate each suspected finding in context. A pattern match is not proof. Never expose secret values; redact them and cite only path/line and safe evidence.
4. If a category cannot be checked confidently or economically, report it as `Needs manual review` rather than silently omitting it.

## Findings

Separate fact from uncertainty. Unconfirmed concerns must say what remains to investigate. Every finding includes repository-relative `file:line`, category, evidence, impact, and severity:

```markdown
## [Security|Performance|Best Practices] Findings
### Critical
- [file:line] — [validated issue and evidence]
### Important
- [file:line] — [issue and impact]
### Minor
- [file:line] — [low-risk issue]
```

Use **Critical** for active vulnerabilities, exposed production secrets, material degradation, or blocking architectural risk; **Important** for meaningful hardening/optimization or compounding violations; **Minor** for low-risk or stylistic items. Finish with counts by category and severity, top priorities, limitations, and: `The developer owns all fix decisions.`

## Safety

Audit is read-only: do not edit, create, delete, install, dispatch, or remediate. Do not contact remote services unless separately authorized by the governing workflow. Preserve uncertainty and evidence in the report.
