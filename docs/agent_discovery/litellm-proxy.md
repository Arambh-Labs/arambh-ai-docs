---
layout: default
title: LiteLLM Proxy Discovery
permalink: /docs/agent-discovery-guide/litellm-proxy/
parent: Agent Discovery Guide
nav_order: 7
---

# LiteLLM Proxy Discovery

Inventory LiteLLM Proxy users, virtual keys, teams, models, credentials, and MCP servers to map LLM gateway blast radius.

**Toolkit name:** `litellm_proxy`

**What discovery finds:** Users, teams, organizations, virtual keys (key aliases treated as agent-like roles), models, credential metadata, and MCP servers. Keys with unrestricted model access (empty allow-list) are linked to a high-sensitivity “all models” surface.

---

## Table of Contents

- [Configuration](#configuration)
- [Required permissions](#required-permissions)
- [Setup instructions](#setup-instructions)
- [Run discovery](#run-discovery)
- [Discovery options](#discovery-options)
- [What you get](#what-you-get)
- [Troubleshooting](#troubleshooting)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Base URL** | String | Yes | LiteLLM Proxy base URL (for example `https://litellm.internal.example`) |
| **API Key** | Password | Yes | Master / admin API key with rights to list users, keys, teams, models, credentials, and MCP |

---

## Required permissions

Use a **master or admin** API key that can call LiteLLM admin APIs such as:

- `/user/list`
- `/key/list`
- `/team/list`
- `/model/info`
- `/v1/mcp/server` (and related credential endpoints)

Network access from Arambh AI to the proxy URL is required.

---

## Setup instructions

### 1. Obtain an admin key

1. On your LiteLLM Proxy deployment, create or copy a master/admin key
2. Confirm the proxy is reachable from the Arambh AI environment

### 2. Configure in Arambh AI

1. Integrations → Litellm Proxy → **Add Configuration**
2. Enter Base URL and API Key
3. Save and run **Healthcheck** (lists users with `max_users=1`)

---

## Run discovery

```json
{
  "toolkit_name": "litellm_proxy",
  "config_name": "default",
  "max_users": 500
}
```

Endpoint: `POST /graphsapi/integrations/blast-radius-discovery`

---

## Discovery options

| Option | Default | Forwarded by HTTP API? | Description |
|--------|---------|------------------------|-------------|
| `max_users` | `500` | Yes | Cap user listing |
| `max_keys` | `500` | No* | Cap virtual key listing |
| `include_teams` | `true` | No* | Include teams |
| `include_organizations` | `true` | No* | Include organizations |
| `include_models` | `true` | No* | Include models |
| `include_credentials` | `true` | No* | Include credential metadata |
| `include_mcp_servers` | `true` | No* | Include MCP servers |

\*These options exist on the integration class; only fields listed in the [parent guide]({{ '/docs/agent-discovery-guide/' | relative_url }}) HTTP allowlist are accepted via `POST .../blast-radius-discovery`. Defaults apply for the rest.

---

## What you get

| Graph element | Examples |
|---------------|----------|
| Root | Proxy host identity |
| Identities | Users, teams, orgs |
| Roles | Virtual keys / agent aliases |
| Surfaces | Models, unrestricted model access |
| Other | Credential metadata, MCP servers |

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| Healthcheck fails | Base URL reachable; API key is master/admin |
| Partial inventory | Key may lack access to keys/teams/MCP endpoints |
| Too many users | Pass `max_users` in the discovery request |

---

**Last Updated:** July 2026
