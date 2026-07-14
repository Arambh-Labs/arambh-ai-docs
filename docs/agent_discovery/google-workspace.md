---
layout: default
title: Google Workspace Discovery
permalink: /docs/agent-discovery-guide/google-workspace/
parent: Agent Discovery Guide
nav_order: 2
---

# Google Workspace Discovery

Inventory third-party OAuth grants across Google Workspace users and map which Workspace surfaces those apps can access.

**Toolkit name:** `google_workspace`

**What discovery finds:** Domain-wide OAuth tokens/grants for users, third-party apps and their scopes, and surfaces such as Drive, Gmail, Calendar, Contacts, Directory, and profile. Apps with dangerous scopes and names matching common AI connectors are highlighted.

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
| **Service Account JSON** | Password | Yes | Full service account key JSON (domain-wide delegation enabled). Stored encrypted. |
| **Admin Email** | String | Yes | Super-admin email to impersonate (for example `admin@company.com`) |
| **Domain Label** | String | No | Label for the root graph node only — does **not** filter which domains are scanned |

---

## Required permissions

1. Enable the **Admin SDK API** on the GCP project that owns the service account.
2. Enable **domain-wide delegation** on the service account.
3. In Google Admin console → Security → Access and data control → API controls → **Domain-wide delegation**, authorize the SA client ID for:

| Scope | Purpose |
|-------|---------|
| `https://www.googleapis.com/auth/admin.directory.user.security` | Read users' OAuth tokens / grants |
| `https://www.googleapis.com/auth/admin.directory.user.readonly` | List users |

The **Admin Email** must be a Workspace **super admin**.

---

## Setup instructions

### 1. Create a service account

1. Google Cloud Console → IAM & Admin → Service Accounts → Create
2. Create a JSON key and download it
3. Enable **Domain-wide delegation** and note the client ID

### 2. Authorize domain-wide delegation

1. Google Admin → Security → API controls → Domain-wide delegation → **Add new**
2. Paste the SA client ID
3. Add the two OAuth scopes listed above
4. Authorize

### 3. Configure in Arambh AI

1. Integrations → Google Workspace → **Add Configuration**
2. Paste the service account JSON and admin email (optional domain label)
3. Save and run **Healthcheck**

---

## Run discovery

```json
{
  "toolkit_name": "google_workspace",
  "config_name": "default",
  "admin_email": "admin@company.com",
  "max_users": null
}
```

`admin_email` in the request body overrides the config value when provided. `max_users` limits how many users are enumerated (`null` / omitted = no limit).

Endpoint: `POST /graphsapi/integrations/blast-radius-discovery`

---

## Discovery options

| Option | Default | Description |
|--------|---------|-------------|
| `admin_email` | from config | Super-admin to impersonate |
| `domain` | from config | Root node label only |
| `max_users` | unlimited | Cap user enumeration |

---

## What you get

| Graph element | Examples |
|---------------|----------|
| Root | Workspace domain identity |
| Identities | Users with grants |
| Roles / apps | Third-party OAuth apps |
| Surfaces | Drive, Gmail, Calendar, Contacts, Directory |
| Edges | User↔app grants, app→scopes / can-read |

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| Healthcheck fails | Admin email is super-admin; Admin SDK enabled; SA JSON valid |
| Delegation / unauthorized_client | Domain-wide delegation client ID and scopes match exactly |
| Empty grants | Users may have no third-party tokens; confirm security scope is authorized |
| Slow or timed out | Use `max_users` for an initial sample scan |

---

**Last Updated:** July 2026
