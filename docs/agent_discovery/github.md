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

---

## Table of Contents

- [Configuration](#configuration)
- [Required permissions](#required-permissions)
- [Setup instructions](#setup-instructions)

---

## Configuration

| Field            | Type     | Required | Description                                                        |
| ---------------- | -------- | -------- | ------------------------------------------------------------------ |
| **API Token**    | Password | Yes      | Org-owner (or equivalent) token with `read:org` / `admin:org:read` |
| **Organization** | String   | Yes      | GitHub organization login (for example `my-company`)               |

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

**Last Updated:** July 2026
