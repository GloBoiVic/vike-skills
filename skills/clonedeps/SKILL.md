---
name: clonedeps
description: Opt-in guidance for copying or cloning dependency source at an explicitly pinned revision; requires a complete target plan and human confirmation.
---

# Clone Dependencies

This skill is inert unless explicitly invoked. It helps plan a controlled dependency-source clone or copy; it never fetches, clones, installs, patches, or changes dependencies automatically.

## When to use

- Use only when the user explicitly requests dependency-source cloning/copying for a named investigation or development purpose.
- Do not use for ordinary dependency installation, automatic vendoring, or unreviewed replacement of a project dependency.

## Required inputs and outputs

Before any operation, record and obtain explicit confirmation for:

- **Target** — the project/repository receiving the dependency and intended purpose.
- **Source** — exact repository or local source, including host when relevant.
- **Revision** — immutable commit, tag, or digest; do not use an unpinned moving reference.
- **Destination** — exact path and whether it exists or contains files.
- **Cleanup** — ownership, retention/removal timing, and treatment of partial output.

Return a plan with the exact operation, expected files, validation, cleanup, and recovery steps. Report the resolved revision and any unknowns.

## Safety boundaries

- Human confirmation is required immediately before every fetch, clone, copy, checkout, or destination-changing command.
- Never request, read, print, store, or transmit credentials, tokens, SSH keys, or private configuration.
- Never install packages, run lifecycle scripts, execute cloned code, alter lockfiles/manifests, or modify project source unless separately requested and confirmed.
- Never overwrite a destination, follow an unpinned revision, or delete partial output without explicit confirmation.
- Prefer local, auditable sources; disclose when a remote source is required.

## Invocation rule

Never invoke this skill automatically. A complete target/source/revision/destination/cleanup plan and explicit human confirmation are prerequisites for any operation.
