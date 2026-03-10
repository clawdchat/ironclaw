# Deploying IronClaw on Coolify

Deploy IronClaw with PostgreSQL (pgvector) using Coolify's Docker Compose support.

## Prerequisites

- A running [Coolify](https://coolify.io) instance (v4+).
- A server connected to Coolify.
- An LLM provider API key (NEAR AI, Anthropic, OpenAI, or any OpenAI-compatible provider).

## Quick Start

### 1. Create a New Resource

1. Open Coolify dashboard → **Projects** → select or create a project.
2. Click **+ Add Resource** → **Docker Compose**.
3. Choose **Git Repository** and point it to this repo, or paste the raw
   `docker-compose.coolify.yml` into the editor.
4. Set the **Docker Compose Location** to `docker-compose.coolify.yml`.

### 2. Configure Environment Variables

In the Coolify **Environment Variables** tab, set the **required** variables:

| Variable | Required | Description |
|---|---|---|
| `POSTGRES_PASSWORD` | **Yes** | Password for the PostgreSQL database. Use a strong random value. |
| `GATEWAY_AUTH_TOKEN` | **Yes** | Bearer token for the web gateway API. Use a strong random value. |
| `NEARAI_API_KEY` | **Yes*** | NEAR AI API key from [cloud.near.ai](https://cloud.near.ai). |

*\*Or configure an alternative LLM provider (see below).*

**Optional variables** — override only if needed:

| Variable | Default | Description |
|---|---|---|
| `AGENT_NAME` | `ironclaw` | Display name for the agent. |
| `NEARAI_MODEL` | `claude-3-5-sonnet-20241022` | NEAR AI model to use. |
| `LLM_BACKEND` | `nearai` | LLM backend (`nearai`, `anthropic`, `openai`, `openai_compatible`). |
| `EMBEDDING_ENABLED` | `false` | Enable semantic memory search (requires embeddings API key). |
| `HEARTBEAT_ENABLED` | `false` | Enable proactive periodic execution. |
| `RUST_LOG` | `ironclaw=info` | Log level. Use `ironclaw=debug` for troubleshooting. |

### 3. Using a Different LLM Provider

Set `LLM_BACKEND` and the corresponding API key:

**Anthropic Direct:**
```
LLM_BACKEND=anthropic
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-sonnet-4-20250514
```

**OpenAI Direct:**
```
LLM_BACKEND=openai
OPENAI_API_KEY=sk-...
```

**OpenAI-Compatible (OpenRouter, Together AI, etc.):**
```
LLM_BACKEND=openai_compatible
LLM_BASE_URL=https://openrouter.ai/api/v1
LLM_API_KEY=sk-or-...
LLM_MODEL=anthropic/claude-sonnet-4
```

### 4. Deploy

Click **Deploy**. Coolify will:

1. Build the IronClaw image from the `Dockerfile`.
2. Start PostgreSQL with pgvector and wait for the health check.
3. Start IronClaw once the database is healthy.

### 5. Access the Web UI

Once deployed, open the URL assigned by Coolify (or configure a custom domain
in the Coolify resource settings). The web gateway listens on port **3000**.

To authenticate API requests, include the token:
```
Authorization: Bearer <your GATEWAY_AUTH_TOKEN>
```

## Health Checks

The IronClaw container includes a built-in Docker `HEALTHCHECK` that polls
`/api/health` every 30 seconds. Coolify will show the container as healthy
once the endpoint responds.

PostgreSQL uses `pg_isready` with a 5-second interval.

## Persistent Data

The `pgdata` Docker volume stores all PostgreSQL data. As long as the volume
is not deleted, your data persists across redeployments.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Container shows **unhealthy** | Check logs — likely a missing or invalid `DATABASE_URL` or LLM key. |
| `Set POSTGRES_PASSWORD` error | Set the `POSTGRES_PASSWORD` environment variable in Coolify. |
| `Set GATEWAY_AUTH_TOKEN` error | Set the `GATEWAY_AUTH_TOKEN` environment variable in Coolify. |
| Database connection refused | Ensure the `postgres` service is healthy before IronClaw starts (handled by `depends_on`). |
| LLM errors | Verify `LLM_BACKEND` and the matching API key are correct. |

For detailed logs, set `RUST_LOG=ironclaw=debug` and redeploy.
