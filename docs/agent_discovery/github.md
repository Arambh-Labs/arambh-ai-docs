---
layout: default
title: GitHub Discovery
permalink: /docs/agent-discovery-guide/github/
parent: Agent Discovery Guide
nav_order: 3
---

# GitHub Discovery

Inventory GitHub Apps, SSO PATs, members, and repositories in an organization to map CI/agent blast radius.

**Toolkit name:** `github`

**What discovery finds:** Org settings, installed GitHub Apps and permissions, SSO credential authorizations (PATs), members and outside collaborators, teams, org secret metadata, repos (collaborators, teams, branch protection), workflows, and OIDC (`id-token: write`) usage. Apps matching agent/CI naming patterns are highlighted.

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
| **API Token** | Password | Yes | Org-owner (or equivalent) token with `read:org` / `admin:org:read` |
| **Organization** | String | Yes | GitHub organization login (for example `my-company`) |

---

## Required permissions

Use a classic PAT (or fine-grained token with equivalent org admin read access) belonging to an **org owner/admin**.

Minimum for discovery:

- `read:org` (toolkit also references `admin:org:read`)

Org admin rights are typically required to list installations and SSO credential authorizations. Discovery reports **declared** permissions and installs — not proof of use.

---

## Setup instructions

### 1. Create a token

1. GitHub → Settings → Developer settings → Personal access tokens
2. Create a token with org-read scopes as above
3. Authorize the token for your org (SSO authorize if SAML SSO is enabled)

### 2. Configure in Arambh AI

1. Integrations → GitHub → **Add Configuration**
2. Paste the API token and organization login
3. Save and run **Healthcheck**

---

## Run discovery

```json
{
  "toolkit_name": "github",
  "config_name": "default",
  "org": "my-company",
  "include_official": true,
  "include_archived": false,
  "team_members": true,
  "max_repos": null
}
```

`org` in the body overrides the config value when provided.

Endpoint: `POST /graphsapi/integrations/blast-radius-discovery`

---

## Discovery options

| Option | Default | Description |
|--------|---------|-------------|
| `org` | from config | Organization login |
| `include_official` | `true` | Include GitHub first-party apps |
| `include_archived` | `false` | Include archived repositories |
| `team_members` | `true` | Include team membership detail |
| `max_repos` | unlimited | Cap repository enumeration |

---

## What you get

| Graph element | Examples |
|---------------|----------|
| Root | Organization identity |
| Roles | GitHub Apps |
| Credentials | SSO PATs (metadata) |
| Identities | Members, collaborators, teams |
| Assets | Repos, workflows, org secrets |
| External | Cloud OIDC consumers where workflows request `id-token` |

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| 401 / 403 | Token valid, SSO-authorized for the org, user is org admin |
| Missing installations | Org admin required for installed apps API |
| Missing SSO PATs | Enterprise/org with SAML SSO; token authorized for SSO |
| Large orgs time out | Set `max_repos` and skip archived repos |

---

**Last Updated:** July 2026
