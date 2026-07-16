---
layout: default
title: Microsoft 365 Discovery
permalink: /docs/agent-discovery-guide/microsoft-365/
parent: Agent Discovery Guide
nav_order: 1
---

# Microsoft 365 Discovery

Inventory Entra ID apps and OAuth grants, and map which Microsoft 365 surfaces those apps can reach.

**Toolkit name:** `microsoft_office365`

---

## Table of Contents

- [Configuration](#configuration)
- [Required permissions](#required-permissions)
- [Setup instructions](#setup-instructions)

---

## Configuration

Add a configuration under **Integrations → Microsoft 365**.

| Field                      | Type     | Required | Description                                                    |
| -------------------------- | -------- | -------- | -------------------------------------------------------------- |
| **Tenant ID**              | String   | Yes      | Entra directory (tenant) ID                                    |
| **Client ID**              | String   | Yes      | App registration application (client) ID                       |
| **Client Secret**          | Password | Yes\*    | Client secret (\*or use certificate auth for discovery)        |
| **Certificate PEM**        | Password | No       | PEM private key (+ cert) for certificate-based app-only auth   |
| **Certificate Thumbprint** | String   | No       | SHA-1 thumbprint if not derived from the PEM                   |
| **User ID**                | String   | No       | Mailbox for email actions — not used by blast radius discovery |
| **Server URL**             | String   | No       | Graph endpoint (default `https://graph.microsoft.com/`)        |

---

## Required permissions

Use an app registration with **application** permissions (admin consent required):

| Permission             | Purpose                                         |
| ---------------------- | ----------------------------------------------- |
| `Application.Read.All` | Read service principals and apps                |
| `Directory.Read.All`   | Read directory objects, grants, and assignments |

**Auth modes:** Client secret **or** certificate JWT assertion (preferred for some tenants). Auth scope: `https://graph.microsoft.com/.default`.

---

## Setup instructions

### 1. Register an application in Entra ID

1. Azure Portal → Microsoft Entra ID → App registrations → **New registration**
2. Name it (for example `Arambh AI - M365 Discovery`)
3. Select **Accounts in this organizational directory only**
4. Register and copy **Application (client) ID** and **Directory (tenant) ID**

### 2. Create credentials

- **Client secret:** Certificates & secrets → New client secret → copy the value once  
  **or**
- **Certificate:** Upload or generate a cert; paste PEM into **Certificate PEM** (and thumbprint if needed)

### 3. Add API permissions

1. API permissions → Add a permission → Microsoft Graph → **Application permissions**
2. Add `Application.Read.All` and `Directory.Read.All`
3. Click **Grant admin consent**

### 4. Configure in Arambh AI

1. Integrations → Microsoft 365 → Configurations → **Add Configuration**
2. Enter Tenant ID, Client ID, and Client Secret (and optional certificate fields)
3. Save, then run **Healthcheck**

---

**Last Updated:** July 2026
