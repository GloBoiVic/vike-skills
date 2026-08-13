---
name: imprint
description: Optionally capture reusable UI visual patterns in a developer-approved registry after building or auditing a component. Ask before creating a missing registry.
---

# Imprint — Optional UI Pattern Capture

Run only when requested. It has no required registry path, `init` dependency, migration policy, or automatic invocation.

```text
/imprint
/imprint [filepath]
/imprint audit
```

Identify the supplied component or recent UI changes; if unclear, ask which component to capture. Read only relevant code and record reusable background, border, radius, text, spacing, interaction, shadow, and accent patterns. Skip context-dependent dimensions, layout, positioning, responsive variants, and one-off animation timing.

Use a registry path already named by the project or developer. If none exists, ask:

`No UI registry is configured. Create a registry at [proposed path]? (yes / no)`

On **no**, report patterns without writing. On **yes**, create only the approved file. Never silently create, migrate, overwrite, or choose a canonical path. Append new entries; update matching entries. Before changing a conflicting pattern, ask which value should win.

Suggested entry:

```markdown
### [Component Name]
File: [filepath]
Last updated: [date]

| Property | Pattern |
| --- | --- |
| Background | [value] |
| Border | [value] |
| Radius | [value] |
| Text | [value] |
| Spacing | [value] |
| Interaction | [value] |

**Pattern notes:** [why it matters and allowed variations]
```

After writing, report the approved path, component, and captured properties. If no registry was approved, say that nothing was written. Flag inconsistencies rather than silently normalizing them.

## Audit

`/imprint audit` is opt-in. Scan existing UI components, report variations and hardcoded values with recommendations, and do not modify components or create/update a registry without explicit approval. An approved baseline may be written only to an approved existing or newly approved path, followed by a deviation list. Audit is not migration.
