---
layout: default
title: Endpoint Discovery
permalink: /docs/agent-discovery-guide/endpoint-discovery/
parent: Agent Discovery Guide
nav_order: 8
---

# Endpoint Discovery

Connect to a collection server that reports local AI harnesses, MCP servers, credentials, and reachability — then build an endpoint blast-radius graph.

**Toolkit name:** `endpoint_discovery`

**What discovery finds** (from collection sensors, typically under `/sensors/observations`):

| Sensor | Content |
|--------|---------|
| **agents** | Local AI harnesses — Claude Code, Codex CLI, Gemini CLI, Cursor, Copilot CLI, opencode, Factory Droid, Hermes, Devin, Claude Cowork, and similar |
| **mcp** | MCP servers attached to harnesses |
| **ghtokens** | GitHub credential presence on the endpoint (**values are never stored**) |
| **reach** | Shell reachability / SSH targets; terminal CLIs may get `inherits-shell` edges |

---

## Table of Contents

- [Configuration](#configuration)
- [Prerequisites](#prerequisites)
- [Setup instructions](#setup-instructions)
- [Run discovery](#run-discovery)
- [Discovery options](#discovery-options)
- [What you get](#what-you-get)
- [Troubleshooting](#troubleshooting)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Collection server BASE URL** | String | Yes | Base URL of the endpoint collection server that exposes sensors |

---

## Prerequisites

1. Deploy (or point at) a **collection server** that Arambh AI can reach.
2. Sensors should report at least **agents**; MCP, GitHub tokens, and reach are optional but enrich the graph.
3. Healthcheck calls the agents sensor — that path must be available.

---

## Setup instructions

### 1. Deploy / locate the collection server

Ensure the collection server is running and publishing observations for the endpoints you care about.

### 2. Configure in Arambh AI

1. Integrations → Endpoint Discovery → **Add Configuration**
2. Enter the **Collection server BASE URL**
3. Save and run **Healthcheck**

---

## Run discovery

```json
{
  "toolkit_name": "endpoint_discovery",
  "config_name": "default"
}
```

Endpoint: `POST /graphsapi/integrations/blast-radius-discovery`

---

## Discovery options

| Option | Default | Forwarded by HTTP API? | Description |
|--------|---------|------------------------|-------------|
| `timeout` | `5.0` (min `0.5`) | No* | Sensor request timeout (seconds) |
| `include_mcp` | `true` | No* | Include MCP sensor data |
| `mcp_project_root` | `None` | No* | Optional MCP project root context |

\*These options exist on the integration class; only fields listed in the [parent guide]({{ '/docs/agent-discovery-guide/' | relative_url }}) HTTP allowlist are accepted via the discovery HTTP endpoint. Defaults apply when calling through the API.

---

## What you get

| Graph element | Examples |
|---------------|----------|
| Root | Endpoint identity (`endpoint_host` in `_meta`) |
| Roles | Agent harnesses (`is_agent: true`), MCP servers |
| Credentials | GitHub token presence (no secrets) |
| Reach | SSH / shell targets |
| Edges | has-harness, has-mcp-server, inherits-shell, can-reach, has-credential |

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| Healthcheck fails | BASE URL reachable; agents sensor responds |
| No MCP nodes | MCP sensor publishing; `include_mcp` default is true when using library defaults |
| Unexpected empty graph | Sensors returning empty observations for that host |

---

**Last Updated:** July 2026
