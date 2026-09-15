---
layout: default
title: Stellar Cyber
permalink: /docs/integrations-guide/stellar-cyber/
parent: Integrations Guide
nav_order: 14
---

# Stellar Cyber

Query and update cases, search raw security events, and manage detections in Stellar Cyber Open XDR.

**Category:** SIEM / Open XDR

**Description:** Stellar Cyber rolls up correlated alerts into scored **cases**. This integration can list and update those cases, pull the alerts behind a specific case, and when a case doesn't give you enough detail, drop down to raw ElasticSearch queries against the underlying event data.

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [List Cases](#1-list-cases)
  - [Get Case Details](#2-get-case-details)
  - [Update Case](#3-update-case)
  - [Search Events](#4-search-events)
  - [Update Event](#5-update-event)
- [Best Practices](#best-practices)
- [FAQ](#faq)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Server URL** | String | Yes | Stellar Cyber DP hostname (e.g., `https://your-instance.stellarcyber.cloud`) |
| **User Email** | String | Yes | Email of the Stellar Cyber user the API key belongs to |
| **API Key** | Password | Yes | Refresh token generated for that user under Users → API Access |
| **Verify SSL** | Checkbox | No | Verify the server's SSL certificate (default: enabled) |

---

## Required Permissions

Stellar Cyber authenticates by exchanging a long-lived API key for a short-lived JWT.

- The user you generate the key for needs **Root scope** and **Super Admin** privileges.
- `search_events` specifically only works with a key made through **Generate New Token**. Keys made through the simpler **Create API Key** option are scoped and can't reach the raw data endpoint — you'll get a 403.
- Case actions (list/get/update) work with either type of key, as long as the user can see the relevant tenant.

---

## Setup Instructions

### 1. Generate an API key in Stellar Cyber

- Log into the Stellar Cyber console
- **System → Administration → Users**, pick the user, click **Edit**
- Open **API Access → Generate New Token**
- Copy the key now — it's shown once

### 2. Note the email and hostname

- **User Email** is whoever's account the key belongs to
- **Server URL** is your DP hostname, e.g. `https://your-instance.stellarcyber.cloud`

### 3. Confirm the key works

```bash
curl -X POST "https://your-instance.stellarcyber.cloud/connect/api/v1/access_token" \
  -H "Authorization: Basic $(echo -n 'user@example.com:YOUR_API_KEY' | base64)" \
  -H "Content-Type: application/x-www-form-urlencoded"
```

You should get back a JSON body with `access_token`.

### 4. Configure in Arambh AI

- Integrations → Stellar Cyber → Configurations tab → **Add Configuration**
- Enter Server URL, User Email, API Key
- Set Verify SSL for your environment
- **Save**
- In the Actions tab, turn on whichever actions you want the platform to use

---

## Supported Actions

### 1. List Cases

Pull cases — Stellar Cyber's aggregated, scored incidents — with optional filters.

**Action:** `stellar_cyber--list_cases`

**Endpoint:** `GET /connect/api/v1/cases`

#### Parameters

- `limit` (optional): max cases to return, up to 500 (default `25`)
- `min_score` (optional): only cases with `case_score` at or above this (0-100)
- `status` (optional): `New`, `In Progress`, `Resolved`, `Cancelled`
- `priority` (optional): `Critical`, `High`, `Medium`, `Low`
- `tenant_id` (optional)
- `assignee` (optional)
- `tags` (optional): comma-separated
- `sort` / `order` (optional): default `case_score` desc
- `config_name` (optional)

#### Example

```json
{
  "min_score": 80,
  "status": "New",
  "limit": 25
}
```

#### Returns

```json
{
  "status": "success",
  "message": "Successfully retrieved cases from Stellar Cyber",
  "data": {
    "total": 171298,
    "cases": [
      {
        "_id": "655459dba3149258cd73014b",
        "name": "Cynet: Data Encrypted for Impact and 133 others",
        "score": 100,
        "status": "New",
        "severity": "Critical",
        "tenant_name": "Acme Corp"
      }
    ]
  }
}
```

---

### 2. Get Case Details

Full detail for one case by ID — score, status, tags, and the alerts rolled into it.

**Action:** `stellar_cyber--get_case_details`

**Endpoint:** `GET /connect/api/v1/cases/{case_id}`

#### Parameters

- `case_id` (required)
- `config_name` (optional)

#### Example

```json
{
  "case_id": "655459dba3149258cd73014b"
}
```

---

### 3. Update Case

Change a case's status, severity, assignee, or tags.

**Action:** `stellar_cyber--update_case`

**Endpoint:** `PUT /connect/api/v1/cases/{case_id}`

#### Parameters

- `case_id` (required)
- `status` (optional): `New`, `In Progress`, `Resolved`, `Cancelled`
- `severity` (optional): `Critical`, `High`, `Medium`, `Low`
- `assignee` (optional): user ID or email
- `tags_to_add` (optional)
- `tags_to_delete` (optional)
- `config_name` (optional)

Needs at least one of the fields above besides `case_id`.

#### Example

```json
{
  "case_id": "655459dba3149258cd73014b",
  "status": "Resolved",
  "tags_to_add": ["reviewed-by-arambh"]
}
```

#### Returns

```json
{
  "_id": "655459dba3149258cd73014b",
  "status": "Resolved",
  "tags": ["reviewed-by-arambh"],
  "modified_at": 1660835991789
}
```

---

### 4. Search Events

Raw ElasticSearch DSL against Stellar Cyber's event indices (default `aella-ser-*`) — for when a case rollup isn't granular enough.

**Action:** `stellar_cyber--search_events`

**Endpoint:** `POST /connect/api/data/{index}/_search`

**Needs:** a key from **Generate New Token** on a Root-scope account — see [Required Permissions](#required-permissions).

#### Parameters

- `index` (optional, default `aella-ser-*`)
- `query` (optional, default `match_all`): ElasticSearch DSL
- `size` (optional, default `100`)
- `config_name` (optional)

#### Example

```json
{
  "index": "aella-ser-*",
  "query": { "query": { "match": { "srcip": "10.0.0.1" } } },
  "size": 50
}
```

#### Returns

```json
{
  "status": "success",
  "message": "Successfully retrieved 12 event(s) from Stellar Cyber",
  "total": 12,
  "data": [ { "_index": "aella-ser-...", "_id": "...", "_source": { "...": "..." } } ]
}
```

---

### 5. Update Event

Set status, add a comment, or tag one event — addressed by index + document ID, which come straight out of a `search_events` hit.

**Action:** `stellar_cyber--update_event`

**Endpoint:** `POST /connect/api/update_ser`

#### Parameters

- `index` (required): e.g. `aella-ser-1610496070892-`
- `event_id` (required): the doc `_id`
- `status` (optional): `New`, `In Progress`, `Ignored`, `Closed`
- `comments` (optional)
- `tag_op` (optional): `add` or `delete`
- `tag` (optional): required if `tag_op` is set
- `config_name` (optional)

Needs at least one of `status`, `comments`, `tag_op`/`tag`.

#### Example

```json
{
  "index": "aella-ser-1610496070892-",
  "event_id": "7PW5SXcBlxPU3jFcFJ_s",
  "status": "Closed",
  "comments": "Reviewed, benign."
}
```

---

## Best Practices

- **Tokens refresh themselves.** JWTs expire 10 minutes after issuance; the integration renews them from the stored API key roughly 30 seconds before expiry, so there's nothing to manage day-to-day.
- **Use cases first, events only when you need to.** `list_cases` / `get_case_details` / `update_case` cover most investigation work. Reach for `search_events` / `update_event` when you need to look at (or triage) the individual detections behind a case.
- **Keep `search_events` queries narrow.** The event indices get large fast — scope the query and set a sane `size` rather than pulling everything back.

---

## FAQ

### Does every action need a Super Admin account?

No — just `search_events`. List/Get/Update Case work with any user who can see the relevant tenant, but the key itself still has to come from a Root-scope Super Admin's **Generate New Token**.

### Why is `search_events` returning a 403?

The key was probably made through **Create API Key** instead of **Generate New Token**, or the user isn't Root-scope/Super Admin. Regenerate it per [Setup Instructions](#setup-instructions).

### Where do I get the `index` / `event_id` for Update Event?

Run `search_events` first — each hit carries `_index` and `_id`, which map directly onto those two fields.

---

**Back to:** [Integrations Overview](../)

**Last Updated:** September 2026
