---
name: clonedeps
description: Opt-in plan for copying dependency source at an immutable revision; requires a complete target plan and confirmation immediately before each risky operation.
---

# Clone Dependencies

Use only for an explicit request to clone/copy named dependency source for a stated purpose. It is not ordinary installation, automatic vendoring, or dependency replacement. It never fetches, clones, installs, patches, executes, or changes dependencies automatically.

## Required plan

Before any operation, specify and obtain confirmation for:

- **Target/purpose** — receiving project and reason.
- **Source** — exact repository or local source and host.
- **Revision** — immutable commit, tag, or digest; never a moving reference.
- **Destination** — exact path and whether it exists or is non-empty.
- **Cleanup/recovery** — ownership, retention/removal, and partial-output handling.

Return exact commands, expected files, validation, cleanup, recovery, resolved revision, and unknowns.

## Safety

Confirm immediately before every fetch, clone, copy, checkout, or destination-changing command. Never read/store/transmit credentials, tokens, keys, or private config; install packages; run lifecycle or cloned code; alter manifests/lockfiles/source; overwrite destinations; or delete partial output without separate explicit confirmation. Prefer local auditable sources and disclose remote use. Never invoke automatically; a complete plan and confirmation are prerequisites.
