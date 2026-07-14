---
layout: default
title: Google Cloud Platform Discovery
permalink: /docs/agent-discovery-guide/google-cloud-platform/
parent: Agent Discovery Guide
nav_order: 4
---

# Google Cloud Platform Discovery

Enumerate IAM bindings and service accounts in a GCP project to map who can reach sensitive cloud surfaces.

**Toolkit name:** `gcp`

**What discovery finds:** Project IAM principals → roles → surfaces (Secret Manager, Storage, Compute, GKE, IAM, BigQuery, Cloud Run, Functions, and related), service accounts, and optional user-managed key metadata. Principals or labels matching AI/ML naming patterns are highlighted. Dangerous roles (owner, editor, SA admin, key admin, token creator, workload identity user) are called out.

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
| **Service Account JSON** | Password | Yes | SA key with read-only IAM access. Stored encrypted; never written to discovery output. |
| **Project ID** | String | Yes | GCP project to enumerate |

---

## Required permissions

Grant the service account read access on the target project, for example:

| Role | Purpose |
|------|---------|
| `roles/viewer` | Broad read of project resources |
| `roles/iam.securityReviewer` | Read IAM policy and related security metadata |

OAuth scope used: `cloud-platform.read-only` (or equivalent read-only cloud-platform access).

Key **material** is never collected — only key metadata when `include_keys` is enabled.

---

## Setup instructions

### 1. Create a read-only service account

1. GCP Console → IAM & Admin → Service Accounts → Create
2. Grant `roles/viewer` and `roles/iam.securityReviewer` on the project
3. Create and download a JSON key

### 2. Configure in Arambh AI

1. Integrations → Google Cloud Platform → **Add Configuration**
2. Paste the service account JSON and project ID
3. Save and run **Healthcheck** (calls `projects.getIamPolicy`)

---

## Run discovery

```json
{
  "toolkit_name": "gcp",
  "config_name": "default",
  "project_id": "my-prod-project",
  "include_keys": true,
  "max_service_accounts": null
}
```

Endpoint: `POST /graphsapi/integrations/blast-radius-discovery`

---

## Discovery options

| Option | Default | Description |
|--------|---------|-------------|
| `project_id` | from config | Project to enumerate |
| `include_keys` | `true` | List user-managed SA key metadata |
| `max_service_accounts` | unlimited | Cap SA enumeration |

---

## What you get

| Graph element | Examples |
|---------------|----------|
| Root | `gcp-project` |
| Identities | Users, groups, service accounts |
| Roles | IAM roles bound on the project |
| Surfaces | Secret Manager, Storage, Compute, GKE, BigQuery, Cloud Run, … |
| Secrets | SA key metadata nodes (when enabled) |

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| Permission denied on IAM policy | SA has `getIamPolicy` / security reviewer on the project |
| Wrong project content | `project_id` matches the project the SA can access |
| Missing key nodes | Set `include_keys: true` |

---

**Last Updated:** July 2026
