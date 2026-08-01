# Decisions

## 2026-08-01 — Replace fixed templates with adaptive context discovery

- **Decision:** Use `/context` with an indexed, selectively loaded knowledge model instead of a fixed project template.
- **Rationale:** Projects vary by domain; adaptive recommendations reduce noise and preserve existing documentation.
- **Constraint:** Initialization never overwrites existing documentation.
