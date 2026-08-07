---
title: Mosaico CLI
sidebar_position: 4
description: Configure connections, inspect resources, and diagnose Mosaico from the command line.
---

The `mosaico` command is included with the Python SDK CLI extra:

```bash
pip install "mosaicolabs[cli]"
```

## Start a project with Codex or Claude

If you have never used Mosaico, you do not need to install anything first. Point
your AI coding tool at the self-contained runbook and let it do the setup:

```text
Read https://docs.mosaico.dev/start.md and follow it to spin up Mosaico in this project.
```

That page covers starting the `mosaicod` daemon, installing the client,
configuring a profile, and verifying the connection with a read-only example.

To leave that guidance in the project so later sessions start with it already
loaded, prepare the current project for AI-assisted Mosaico development:

```bash
mosaico ai init
```

Without a prior install, run it directly:

```bash
uvx --from "mosaicolabs[cli]" mosaico ai init
```

The command creates a shared `.mosaico/AI_GUIDE.md` and adds a managed section
to the native project instructions for both Codex (`AGENTS.md`) and Claude
(`CLAUDE.md`). The Codex section is self-contained; Claude imports the shared
guide using its native project-instruction syntax. Existing instructions outside
the managed section are preserved, and rerunning the command refreshes the guide
without duplicating it. It does not store credentials or require a configured
Mosaico connection.

The generated onboarding guide directs agents to
[`llms.txt`](https://docs.mosaico.dev/llms.txt) and treats that navigable index
and its linked pages as the source of truth for current Mosaico behavior.

Then start either tool in the project and ask:

```text
Spin up Mosaico in this project.
```

Prepare only one AI coding system when needed:

```bash
mosaico ai init --assistant codex
mosaico ai init --assistant claude
```

For automation, setup results are available as versioned structured output:

```bash
mosaico ai init --output json
```

## Configure a profile

Interactive setup:

```bash
mosaico profile add local
```

Non-interactive setup:

```bash
mosaico profile add local \
  --no-interactive \
  --host localhost \
  --port 6726 \
  --default
```

Profile files are written with owner-only permissions on POSIX systems. Prefer the `MOSAICO_API_KEY` environment variable when credentials should not be stored locally.

## Diagnose a connection

```bash
mosaico doctor
```

The command checks the resolved profile, configuration permissions, TLS certificate path, DNS, and TCP connectivity. It never prints the API-key value.

For automation and AI development tools, request structured output:

```bash
mosaico doctor --output json
mosaico profile ls --output json
mosaico sequence ls --output jsonl
mosaico topic ls --output csv
```

JSON collection documents include a `schema_version`. Scripts should inspect that field before depending on the document structure.

Skip network checks when validating only local configuration:

```bash
mosaico doctor --no-network
```

The command exits non-zero when a required check fails. Warnings, such as overly broad configuration permissions, are reported without hiding otherwise useful diagnostics.
