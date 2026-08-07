---
name: mosaico-onboard
description: Bootstrap a project from no Mosaico installation to a verified daemon and client connection. Use when a user says "start using Mosaico", "set up Mosaico", "install Mosaico", "onboard this project to Mosaico", or asks for a first working Mosaico integration in Codex or Claude.
---

# Mosaico Onboard

Take the current project from zero Mosaico components to a working, verified
connection. Keep installation choices environment-aware and obtain current
commands from Mosaico documentation instead of model memory.

## Source of truth

Fetch `https://docs.mosaico.dev/llms.txt` before making changes. Navigate from
that index to the relevant Getting Started, daemon setup, client, and CLI pages.
Treat the index and its linked pages as authoritative for current packages,
commands, versions, and platform support.

If the documentation cannot be reached, ask the user to enable access to
`docs.mosaico.dev`; do not substitute remembered commands. If linked pages
conflict, identify the conflict and stop rather than guessing.

## Workflow

1. Inspect the operating system, architecture, project, dependency manager,
   container runtime, Python environment, occupied ports, and existing Mosaico
   components. Do not mutate the system during inspection.
2. Select the simplest supported path from the current documentation. Default to
   the official local container setup unless the user supplies a remote daemon.
   Use stable releases and official Mosaico, GitHub, PyPI, and GHCR sources only.
3. Explain the selected path briefly. Ask for approval only when an OS-level
   dependency, administrator access, GUI action, restart, or expanded network
   access is required. Continue autonomously after approval.
4. Start `mosaicod` with persistent storage and bind it to localhost only. Wait
   for readiness. Never enable unauthenticated non-loopback access.
5. Install the supported client SDK and CLI using the project's dependency
   conventions. Avoid global language-package installation unless requested.
   Configure a local profile without placing credentials in source files,
   command arguments, logs, or chat.
6. Run `mosaico doctor --output json` and inspect `schema_version`, status, and
   checks. Diagnose failures before changing application code.
7. Add and run the smallest read-only connection example supported by the
   project's language. Listing sequences is sufficient. Do not ingest or delete
   data merely to prove connectivity.
8. Report installed versions, changed files, daemon and data locations,
   validation evidence, restart and non-destructive stop commands, and any
   remaining approval or security work.

## Guardrails

- Never expose API keys, authorization metadata, or private-key material.
- Never bind a prototype daemon beyond loopback without explicit authorization
  and the documented TLS and authentication setup.
- Never remove volumes or existing data during onboarding.
- Never overwrite an existing project configuration without preserving it or
  obtaining explicit approval.
- Never claim completion until both daemon readiness and a real client operation
  have been verified.
