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

---

## Table of Contents

- [Configuration](#configuration)
- [Required permissions](#required-permissions)
- [Setup instructions](#setup-instructions)

---

## Configuration

| Field                    | Type     | Required | Description                                                                            |
| ------------------------ | -------- | -------- | -------------------------------------------------------------------------------------- |
| **Service Account JSON** | Password | Yes      | SA key with read-only IAM access. Stored encrypted; never written to discovery output. |
| **Project ID**           | String   | Yes      | GCP project to enumerate                                                               |

---

## Required permissions

Grant the service account read access on the target project, for example:

| Role                         | Purpose                                       |
| ---------------------------- | --------------------------------------------- |
| `roles/viewer`               | Broad read of project resources               |
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

**Last Updated:** July 2026
