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

---

## Table of Contents

- [Configuration](#configuration)
- [Required permissions](#required-permissions)
- [Setup instructions](#setup-instructions)

---

## Configuration

| Field                    | Type     | Required | Description                                                                        |
| ------------------------ | -------- | -------- | ---------------------------------------------------------------------------------- |
| **Service Account JSON** | Password | Yes      | Full service account key JSON (domain-wide delegation enabled). Stored encrypted.  |
| **Admin Email**          | String   | Yes      | Super-admin email to impersonate (for example `admin@company.com`)                 |
| **Domain Label**         | String   | No       | Label for the root graph node only — does **not** filter which domains are scanned |

---

## Required permissions

1. Enable the **Admin SDK API** on the GCP project that owns the service account.
2. Enable **domain-wide delegation** on the service account.
3. In Google Admin console → Security → Access and data control → API controls → **Domain-wide delegation**, authorize the SA client ID for:

| Scope                                                           | Purpose                           |
| --------------------------------------------------------------- | --------------------------------- |
| `https://www.googleapis.com/auth/admin.directory.user.security` | Read users' OAuth tokens / grants |
| `https://www.googleapis.com/auth/admin.directory.user.readonly` | List users                        |

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

**Last Updated:** July 2026
