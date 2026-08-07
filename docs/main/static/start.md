# Spin up Mosaico

> Self-contained runbook for an AI coding system. Everything needed to go from an
> empty machine to a verified Mosaico connection is on this page. Follow it in
> order and stop at the first step that fails.

Mosaico is a data platform for robotics and physical AI. It has two parts: the
`mosaicod` daemon (server, stores and serves data) and a client SDK (`mosaicolabs`,
Python). Nothing works until a daemon is reachable.

This page describes a **local development** setup. Do not use this configuration
for production or for shared machines; see
<https://docs.mosaico.dev/docs/daemon/install.md> for deployment options.

## Rules for the agent following this page

- Run one step at a time and show the developer the real command output.
- Do not print, echo, commit, or hard-code API keys. Local development needs none.
- Do not invent SDK methods, topic schemas, profile names, hosts, or ports. If
  something is not on this page, read <https://docs.mosaico.dev/llms.txt> and
  navigate to the linked page that covers it.
- Do not write data, define ontologies, or change project configuration until the
  read-only check in step 5 succeeds.
- Parse `--output json`, never a formatted table.

## Step 1 — check the preconditions

```bash
docker --version && docker compose version && python3 --version
```

Docker is required to run the daemon. Python 3.10 or newer is required for the
SDK. If Docker is unavailable, stop and offer the binary install path at
<https://docs.mosaico.dev/docs/daemon/install.md> instead of improvising.

## Step 2 — start the daemon

Write `compose.yaml` in the project directory (do not overwrite an existing
`compose.yaml` or `docker-compose.yml` — ask the developer first):

```yaml title="compose.yaml"
services:
  db:
    image: postgres:18
    environment:
      POSTGRES_HOST_AUTH_METHOD: trust

  mosaicod:
    image: ghcr.io/mosaico-labs/mosaicod:latest
    environment:
      MOSAICOD_DB_URL: postgresql://postgres@db:5432/postgres
      MOSAICOD_STORE_ENDPOINT: file:///tmp
      MOSAICOD_STORE_BUCKET: mosaico
    command: run --host 0.0.0.0
    depends_on:
      - db
    ports:
      - "6726:6726"
```

Data lands in `/tmp/mosaico` inside the container. Change
`MOSAICOD_STORE_ENDPOINT` to persist it elsewhere. The daemon listens on port
`6726`. The full environment reference is at
<https://docs.mosaico.dev/docs/daemon/env.md>.

Start it:

```bash
docker compose up -d
```

Confirm both services are up before continuing:

```bash
docker compose ps
```

## Step 3 — install the client

The SDK is the `mosaicolabs` package; the `mosaico` CLI ships in its `cli` extra.
Match the project's existing dependency manager rather than defaulting to `pip`:

```bash
pip install "mosaicolabs[cli]"
```

```bash
uv add "mosaicolabs[cli]"
```

To run the CLI once without installing it into the project:

```bash
uvx --from "mosaicolabs[cli]" mosaico --help
```

## Step 4 — configure and verify the connection

Create a default local profile:

```bash
mosaico profile add local --no-interactive --host localhost --port 6726 --default
```

Then run diagnostics and read the JSON:

```bash
mosaico doctor --output json
```

Inspect `schema_version`, the top-level `status`, and each entry in `checks`.
The command exits non-zero when a required check fails, and it never prints the
API-key value. If the `tcp` check fails, the daemon is not up yet — re-check
`docker compose ps` and `docker compose logs mosaicod`.

For a remote or secured daemon, do not put a key in source. Set it in the
environment instead:

```bash
export MOSAICO_API_KEY=...
```

Key management is documented at <https://docs.mosaico.dev/docs/daemon/api_key.md>
and TLS at <https://docs.mosaico.dev/docs/daemon/tls.md>.

## Step 5 — run the smallest read-only example

```py title="mosaico_check.py"
from mosaicolabs import MosaicoClient

# Connect to the Mosaico server
with MosaicoClient.connect(host="localhost", port=6726) as client:
    # List available sequences
    sequences = client.list_sequences()
    print(f"Connected! Found sequences: {sequences}")
```

On a fresh daemon this prints an empty list — that is success, not a failure.
The equivalent check from the CLI:

```bash
mosaico sequence ls --output json
mosaico topic ls --output json
```

Report the result to the developer before proposing any write, ingestion,
schema, or production change.

## Step 6 — go from here

Concepts, in the order they usually matter:

- **Sequence** — one recording session. **Topic** — one sensor channel inside it.
  **Ontology** — the typed schema describing what a message actually contains.
  Overview: <https://docs.mosaico.dev/docs/overview.md>
- Write your first data: <https://docs.mosaico.dev/docs/learn/writing_single_topic.md>
- Query the catalog: <https://docs.mosaico.dev/docs/learn/query_sequences.md>
- Stream data back out: <https://docs.mosaico.dev/docs/learn/streaming_data.md>
- Data for training pipelines: <https://docs.mosaico.dev/python-sdk/SDK/bridges/ml/>
- Full navigable index for agents: <https://docs.mosaico.dev/llms.txt>

To leave this guidance in the project so future sessions start with it:

```bash
mosaico ai init
```

## Shutting down

```bash
docker compose down
```

Add `-v` only if the developer explicitly asks to discard the stored data.
