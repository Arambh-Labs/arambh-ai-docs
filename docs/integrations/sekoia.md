---
layout: default
title: Sekoia XDR
permalink: /docs/integrations-guide/sekoia/
parent: Integrations Guide
nav_order: 15
---

# Sekoia XDR

List, read, and update alerts and cases in Sekoia.io XDR, and leave comments on either.

**Category:** SIEM / XDR

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [List Alerts](#1-list-alerts)
  - [Get Alert Details](#2-get-alert-details)
  - [Update Alert Status](#3-update-alert-status)
  - [Add Alert Comment](#4-add-alert-comment)
  - [List Cases](#5-list-cases)
  - [Get Case Details](#6-get-case-details)
  - [Update Case](#7-update-case)
  - [Add Case Comment](#8-add-case-comment)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Region** | Select | Yes | Where your Sekoia workspace is hosted: FRA1, FRA2, MCO1, UAE1, USA1 or SGP1 (default: FRA1) |
| **API Key** | Password | Yes | API key created under Settings → Workspace → API Keys |

The region decides which API host we call:

| Region | API base URL |
|--------|--------------|
| FRA1 | `https://api.sekoia.io` |
| FRA2 | `https://app.fra2.sekoia.io/api` |
| MCO1 | `https://app.mco1.sekoia.io/api` |
| UAE1 | `https://app.uae1.sekoia.io/api` |
| USA1 | `https://app.usa1.sekoia.io/api` |
| SGP1 | `https://app.sgp1.sekoia.io/api` |

If you're not sure which one you're on, look at the URL you log in with. `app.sekoia.io` is FRA1; the others have the region in the hostname.

---

## Required Permissions

Sekoia uses the API key as a bearer token on every request (`Authorization: Bearer <key>`). There's no token exchange or refresh - the key works until it expires (if you set an expiry) or someone revokes it.

Only a workspace admin can create API keys. Permissions are set on the key itself, so tick the ones that cover the actions you plan to turn on:

| Permission | Needed for |
|------------|------------|
| View alerts | List Alerts, Get Alert Details, Update Alert Status, and the connection test |
| Update alert status | Update Alert Status |
| Comment on alerts | Add Alert Comment |
| View cases | List Cases, Get Case Details |
| Manage cases | Update Case, Add Case Comment |
| Comment on cases | Add Case Comment |

If you only want Arambh AI to read, the two "View" permissions are enough. Sekoia's own guidance is to start read-only and add write access later.

See Sekoia's [API documentation](https://docs.sekoia.com/developer/api/) for the full reference.

---

## Setup Instructions

### 1. Create an API key in Sekoia

- Log into Sekoia as a workspace admin
- **Settings → Workspace → API Keys**, click **+ API key**
- Name it something you'll recognise later (e.g. `arambh-ai`)
- Add a description - Sekoia wants at least 10 characters (e.g. "Arambh AI alert and case triage")
- Set an expiry: 30, 180 or 365 days, a custom date up to a year out, or no expiration
- Tick the permissions listed above
- **Save**, then copy the key straight away - Sekoia only shows it once

### 2. Find your region

Check the hostname you log in with (see the table under [Configuration](#configuration)).

### 3. Confirm the key works

```bash
curl "https://api.sekoia.io/v1/me" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Swap in your region's base URL if you're not on FRA1. A working key returns your profile as JSON. A `401` with `"The token is invalid"` means the key is wrong, expired or revoked.

### 4. Configure in Arambh AI

- Integrations → Sekoia XDR → Configurations tab → **Add Configuration**
- Pick your Region and paste the API Key
- **Save** - the connection test pulls one alert, so it also checks the key can view alerts
- In the Actions tab, turn on whichever actions you want the platform to use

When the key expires, create a new one and paste it into the same configuration. Nothing else needs changing.

---

## Supported Actions

Every alert and case action takes either the UUID or the short ID you see in the Sekoia UI (`AL…` for alerts, `CA…` for cases). Short IDs get looked up first, which costs one extra API call.

### 1. List Alerts

Pull alerts with optional filters. Newest first by default.

**Action:** `sekoia--list_alerts`

**Endpoint:** `GET /v1/sic/alerts`

**Permissions:** View alerts

#### Parameters

- `limit` (optional): max alerts to return, up to 100 (default `25`)
- `offset` (optional): for paging (default `0`)
- `status_name` (optional): e.g. `Pending`, `Acknowledged`, `Ongoing`, `Closed`, `Rejected`. Comma-separate to match any of several
- `rule_name` (optional): name of the detection rule that fired. Comma-separated
- `entity_name` (optional): comma-separated
- `short_id` (optional): comma-separated
- `min_urgency` (optional): only alerts with urgency at or above this (0-100)
- `created_after` (optional): Krafter AI time template (e.g. `{{now - 24h}}`) or ISO-8601 (e.g. `2026-10-01T00:00:00Z`)
- `created_before` (optional): same formats. Defaults to now when `created_after` is set
- `sort` (optional): `created_at`, `updated_at`, `urgency`, `status`, `rule_name`, `source`, `target`, `entity`, `type`, `similar` (default `created_at`)
- `direction` (optional): `asc` or `desc` (default `desc`)
- `config_name` (optional)

Sekoia's urgency bands are Low (0-49), Medium (50-74) and High (75-100), so `min_urgency: 75` gets you the high ones.

#### Example

```json
{
  "status_name": "Pending",
  "min_urgency": 75,
  "created_after": "{{now - 24h}}",
  "limit": 25
}
```

#### Returns

```json
{
  "status": "success",
  "message": "Successfully retrieved alerts from Sekoia",
  "total": 3,
  "data": [
    {
      "uuid": "1b4d2c6e-...",
      "short_id": "ALe8kQ2xV9pT",
      "title": "Suspicious PowerShell download cradle",
      "created_at": 1792310400,
      "...": "..."
    }
  ]
}
```

---

### 2. Get Alert Details

Everything on one alert - comments, history and countermeasures come back by default. STIX and linked cases are optional.

**Action:** `sekoia--get_alert_details`

**Endpoint:** `GET /v1/sic/alerts/{uuid}`

**Permissions:** View alerts

#### Parameters

- `alert_id` (required): UUID or short ID
- `include_stix` (optional): add the STIX bundle with observables and indicators (default `false`)
- `include_cases` (optional): add the cases the alert belongs to (default `true`)
- `config_name` (optional)

#### Example

```json
{
  "alert_id": "ALe8kQ2xV9pT",
  "include_stix": true
}
```

---

### 3. Update Alert Status

Moves an alert to a new status, with an optional comment.

**Action:** `sekoia--update_alert_status`

**Endpoint:** `GET /v1/sic/alerts/{uuid}/workflow`, then `PATCH /v1/sic/alerts/{uuid}/workflow`

**Permissions:** View alerts (to list the alert's actions) and Update alert status (to apply one)

Sekoia doesn't take a status name here - it wants the UUID of one of the actions currently available on that alert. So we fetch the alert's available actions first, match yours by name (case doesn't matter) or UUID, and send that. If nothing matches, the error lists what the alert does allow so the platform can try again.

What the actions are called depends on the workspace. The default statuses are Pending, Acknowledged, Ongoing, Closed and Rejected, but many workspaces have their own.

#### Parameters

- `alert_id` (required): UUID or short ID
- `action` (required): action name as Sekoia shows it (e.g. `Acknowledge`, or a target status like `Closed`), or the action UUID
- `comment` (optional): why you're changing it
- `config_name` (optional)

#### Example

```json
{
  "alert_id": "ALe8kQ2xV9pT",
  "action": "Closed",
  "comment": "Admin running a scheduled script. Not malicious."
}
```

#### When the action doesn't match

```json
{
  "tool_call_status": "fail",
  "error": "Action 'Resolve' is not available for alert ALe8kQ2xV9pT. Available actions: [{'id': '...', 'name': 'Acknowledged'}, {'id': '...', 'name': 'Closed'}]"
}
```

---

### 4. Add Alert Comment

Leaves a note on an alert.

**Action:** `sekoia--add_alert_comment`

**Endpoint:** `POST /v1/sic/alerts/{uuid}/comments`

**Permissions:** Comment on alerts. Also View alerts if you pass a short ID, since we look up the UUID first

#### Parameters

- `alert_id` (required): UUID or short ID
- `content` (required): the comment
- `author` (optional): name to show on the comment
- `config_name` (optional)

#### Example

```json
{
  "alert_id": "ALe8kQ2xV9pT",
  "content": "Source IP belongs to the internal vuln scanner. Expected traffic.",
  "author": "Arambh AI"
}
```

---

### 5. List Cases

Pull cases - groups of related alerts someone is working on - with optional filters.

**Action:** `sekoia--list_cases`

**Endpoint:** `GET /v1/sic/cases`

**Permissions:** View cases

#### Parameters

- `limit` (optional): max cases to return (default `25`)
- `offset` (optional): for paging (default `0`)
- `status_name` (optional): e.g. `Open`, `Closed`. Comma-separated
- `priority` (optional): `low`, `medium`, `high`
- `tags` (optional): comma-separated
- `assignee` (optional): assignee UUID. Comma-separated
- `title` (optional)
- `created_after` (optional): Krafter AI time template (e.g. `{{now - 7d}}`) or ISO-8601
- `created_before` (optional): same formats. Defaults to now when `created_after` is set
- `sort` (optional): field to sort by (default `created_at`)
- `direction` (optional): `asc` or `desc` (default `desc`)
- `config_name` (optional)

#### Example

```json
{
  "priority": "high",
  "created_after": "{{now - 7d}}"
}
```

#### Returns

```json
{
  "status": "success",
  "message": "Successfully retrieved cases from Sekoia",
  "total": 1,
  "data": [
    {
      "uuid": "9a7f3e21-...",
      "short_id": "CAm4Rt7nWq2Z",
      "title": "Credential stuffing against VPN gateway",
      "priority": "high",
      "status": "Open",
      "tags": ["vpn"],
      "...": "..."
    }
  ]
}
```

---

### 6. Get Case Details

Full detail for one case - status, priority, tags, verdict - plus the alerts attached to it.

**Action:** `sekoia--get_case_details`

**Endpoint:** `GET /v1/sic/cases/{case_uuid}` and `GET /v1/sic/cases/{case_uuid}/alerts`

**Permissions:** View cases

#### Parameters

- `case_id` (required): UUID or short ID
- `include_alerts` (optional): also return the case's alerts (default `true`)
- `alerts_limit` (optional): how many of those alerts to return (default `50`)
- `config_name` (optional)

If the case loads but its alerts don't, you still get the case back, with an `alerts_error` field explaining what went wrong.

#### Example

```json
{
  "case_id": "CAm4Rt7nWq2Z"
}
```

---

### 7. Update Case

Change a case's status, priority, title, description, tags or verdict analysis.

**Action:** `sekoia--update_case`

**Endpoint:** `PATCH /v1/sic/cases/{case_uuid}`

**Permissions:** Manage cases. Also View cases if you pass a short ID or change tags, since both read the case first

#### Parameters

- `case_id` (required): UUID or short ID
- `status_name` (optional): e.g. `Open`, `Closed`
- `priority` (optional): `low`, `medium`, `high`
- `title` (optional)
- `description` (optional)
- `verdict_analysis` (optional): free text
- `tags_to_add` (optional)
- `tags_to_delete` (optional)
- `config_name` (optional)

Needs at least one field besides `case_id`.

Sekoia overwrites the whole tag list on update. When you pass `tags_to_add` or `tags_to_delete`, we read the case's current tags first and send back the merged list, so existing tags aren't lost.

#### Example

```json
{
  "case_id": "CAm4Rt7nWq2Z",
  "status_name": "Closed",
  "tags_to_add": ["reviewed-by-arambh"],
  "verdict_analysis": "Blocked at the gateway. No successful logins from the source range."
}
```

---

### 8. Add Case Comment

Leaves a note on a case.

**Action:** `sekoia--add_case_comment`

**Endpoint:** `POST /v1/sic/cases/{case_uuid}/comments`

**Permissions:** Comment on cases and Manage cases - Sekoia asks for both. Also View cases if you pass a short ID

#### Parameters

- `case_id` (required): UUID or short ID
- `content` (required): the comment
- `config_name` (optional)

#### Example

```json
{
  "case_id": "CAm4Rt7nWq2Z",
  "content": "All 14 alerts trace back to the same /24. Range blocked on the edge firewall."
}
```

---

**Back to:** [Integrations Overview](../)

**Last Updated:** October 2026
