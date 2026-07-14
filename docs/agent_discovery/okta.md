---
layout: default
title: Okta Discovery
permalink: /docs/agent-discovery-guide/okta/
parent: Agent Discovery Guide
nav_order: 5
---

# Okta Discovery

Inventory Okta OIDC applications and authorization-server policies that grant OAuth scopes — useful for finding AI/agent apps registered through SSO.

**Toolkit name:** `okta`

**What discovery finds:** OIDC apps registered under the org, custom authorization servers (policies and rules), granted-scope edges, and apps whose names look like AI agents (Claude, Anthropic, OpenAI/GPT patterns).

---

## Table of Contents

- [Configuration](#configuration)
- [Required permissions](#required-permissions)
- [Setup instructions](#setup-instructions)
- [Run discovery](#run-discovery)
- [What you get](#what-you-get)
- [Troubleshooting](#troubleshooting)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Base URL** | String | Yes | Okta org URL (for example `https://your-org.okta.com`) |
| **Client ID** | String | Yes | API Services application client ID |
| **Private JWK** | Password | Yes | Private JWK for `private_key_jwt` client authentication |

Optional advanced: you can override OAuth scopes in stored config (`scope`) if your org requires a custom set. Default scopes:

```
okta.apps.read okta.authorizationServers.read okta.orgs.read
```

---

## Required permissions

Create an **API Services** application in Okta with:

- Client authentication: **Public key / Private key** (`private_key_jwt`)
- Granted scopes: `okta.apps.read`, `okta.authorizationServers.read`, `okta.orgs.read` (or your custom `scope` string)

---

## Setup instructions

### 1. Create an API Services app in Okta

1. Okta Admin → Applications → Create App Integration → **API Services**
2. Generate a key pair; store the **private JWK** securely
3. Grant the read scopes listed above

### 2. Configure in Arambh AI

1. Integrations → Okta → **Add Configuration**
2. Enter Base URL, Client ID, and Private JWK
3. Save and run **Healthcheck** (token fetch + `GET /api/v1/apps?limit=1`)

---

## Run discovery

```json
{
  "toolkit_name": "okta",
  "config_name": "default"
}
```

Endpoint: `POST /graphsapi/integrations/blast-radius-discovery`

> Note: An in-code option `max_apps` (default `500`) exists on the integration class but is not currently forwarded by the HTTP allowlist. Prefer configuring via the integration API body fields documented in the [parent guide]({{ '/docs/agent-discovery-guide/' | relative_url }}).

---

## What you get

| Graph element | Examples |
|---------------|----------|
| Root | Okta org identity |
| Roles | OIDC applications (`client_id` keyed) |
| Authz | Custom authorization servers |
| Scopes | OAuth scopes granted via policies |
| Edges | registers, hosts, oauth-scope |

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| Token / JWT errors | Private JWK matches the public key registered on the API Services app |
| 403 on apps | `okta.apps.read` (and related) scopes granted |
| Missing authz servers | Org has custom authorization servers; `okta.authorizationServers.read` granted |

---

**Last Updated:** July 2026
