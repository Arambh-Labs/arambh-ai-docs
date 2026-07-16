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

---

## Table of Contents

- [Configuration](#configuration)
- [Required permissions](#required-permissions)
- [Setup instructions](#setup-instructions)

---

## Configuration

| Field        | Type     | Required | Description                                                                                 |
| ------------ | -------- | -------- | ------------------------------------------------------------------------------------------- |
| **Base URL** | String   | Yes      | LiteLLM Proxy base URL (for example `https://litellm.internal.example`)                     |
| **API Key**  | Password | Yes      | Master / admin API key with rights to list users, keys, teams, models, credentials, and MCP |

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

**Last Updated:** July 2026
