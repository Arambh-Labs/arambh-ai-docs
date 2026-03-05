---
layout: default
title: SentinelOne
permalink: /docs/integrations-guide/sentinel-one/
parent: Integrations Guide
nav_order: 5
---

# SentinelOne

Integration with SentinelOne EDR platform for endpoint threat detection, investigation, and response.

**Category:** EDR (Endpoint Detection and Response)

**Description:** Retrieve threats, investigate file hashes, run Deep Visibility queries, and fetch cloud detection alerts from SentinelOne.

---

## Table of Contents
- [Configuration](#configuration)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [Get Hash Details](#1-get-hash-details)
  - [Get Threat Details](#2-get-threat-details)
  - [Fetch Threats](#3-fetch-threats)
  - [Get Threat Events](#4-get-threat-events)
  - [Get Events](#5-get-events)
  - [Get Events By Type](#6-get-events-by-type)
  - [Get Alerts](#7-get-alerts)
- [Best Practices](#best-practices)
- [FAQ](#faq)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Base URL** | String | Yes | SentinelOne management console URL (e.g., `https://your-tenant.sentinelone.net`) |
| **API Token** | Password | Yes | SentinelOne API token generated from the management console |
| **API Version** | String | Yes | SentinelOne API version (default: `v2.1`) |

---

## Setup Instructions

### 1. Generate an API Token

1. Log into your SentinelOne management console
2. Navigate to **Settings** → **Users**
3. Select your user account (or create a service account)
4. Click **Generate API Token**
5. Copy the token immediately — it will not be shown again

> **Note:** API tokens are tied to the user's role and permissions. Use a dedicated service account with least-privilege access.

### 2. Configure in Arambh AI

- Go to **Settings** → **Integrations** → **SentinelOne**
- Enter your **Base URL** (e.g., `https://your-tenant.sentinelone.net`)
- Paste your **API Token**
- Set the **API Version** (default: `v2.1`)
- Click **Save** to save the configuration
- In the **Actions** tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Get Hash Details

Get the reputation details of a file hash from SentinelOne.

**Action:** `sentinel_one--get_hash_details`

#### Parameters

- `hash_id` (required): The SHA1 or SHA256 hash to look up
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "hash_id": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
}
```

#### Returns

- Reputation data for the hash, including threat classification and verdict

---

### 2. Get Threat Details

Get details of one or more SentinelOne threats by their IDs.

**Action:** `sentinel_one--get_threat_details`

#### Parameters

- `ids` (required): Comma-separated list of threat IDs to retrieve
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "ids": "1234567890,9876543210"
}
```

#### Returns

- Array of threat objects including classification, agent, file details, and status

---

### 3. Fetch Threats

Fetch all threats from SentinelOne with optional filters. Automatically paginates through all results.

**Action:** `sentinel_one--fetch_threats`

#### Parameters

- `limit` (optional): Maximum number of threats per page (default: `100`, max: `1000`)
- `created_after` (optional): Filter threats created after this ISO 8601 datetime (e.g., `2024-01-01T00:00:00Z`)
- `created_before` (optional): Filter threats created before this ISO 8601 datetime
- `resolved` (optional): Filter by resolved status (`true` or `false`)
- `agent_id` (optional): Filter threats by agent ID
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "created_after": "2026-03-01T00:00:00Z",
  "resolved": false
}
```

#### Returns

- Full list of threat objects across all pages, including threat ID, classification, agent info, file details, and timestamps

---

### 4. Get Threat Events

Get the list of events associated with a specific SentinelOne threat.

**Action:** `sentinel_one--get_threat_events`

#### Parameters

- `threat_id` (required): The SentinelOne threat ID to retrieve events for
- `event_types` (optional): Comma-separated list of event types to filter (e.g., `dns,ip`)
- `limit` (optional): Maximum number of events to return (default: `50`)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "threat_id": "1234567890",
  "event_types": "dns,ip",
  "limit": 100
}
```

#### Returns

- Array of events associated with the threat, including event type, timestamp, and related metadata

---

### 5. Get Events

Run a Deep Visibility query on SentinelOne and return all matching events.

**Action:** `sentinel_one--get_events`

#### Parameters

- `query` (required): Deep Visibility PowerQuery string (e.g., `EventType = "Process Creation"`)
- `from_date` (required): Query start time in ISO 8601 format (e.g., `2024-01-01T00:00:00.000Z`)
- `to_date` (required): Query end time in ISO 8601 format (e.g., `2024-01-02T00:00:00.000Z`)
- `query_type` (optional): Comma-separated query types (e.g., `events,crossEvents`)
- `site_ids` (optional): Comma-separated site IDs to scope the query
- `account_ids` (optional): Comma-separated account IDs to scope the query
- `limit` (optional): Maximum number of events to return (default: `100`)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "query": "EventType = \"Process Creation\" AND SrcProcName = \"powershell.exe\"",
  "from_date": "2026-03-01T00:00:00.000Z",
  "to_date": "2026-03-02T00:00:00.000Z",
  "limit": 200
}
```

#### Returns

- Array of events matching the query, along with pagination metadata

#### Query Lifecycle

The action creates the Deep Visibility query, polls for completion every 20 seconds (up to 10 minutes), then fetches results.

---

### 6. Get Events By Type

Run a Deep Visibility query on SentinelOne and return events filtered by a specific event type.

**Action:** `sentinel_one--get_events_by_type`

#### Parameters

- `query` (required): Deep Visibility PowerQuery string
- `from_date` (required): Query start time in ISO 8601 format
- `to_date` (required): Query end time in ISO 8601 format
- `event_type` (required): Event type to retrieve. Supported values: `dns`, `ip`, `url`, `file`, `process`, `registry`, `scheduled_task`, `network_actions`, `login`, `logout`
- `query_type` (optional): Comma-separated query types (e.g., `events,crossEvents`)
- `site_ids` (optional): Comma-separated site IDs to scope the query
- `account_ids` (optional): Comma-separated account IDs to scope the query
- `limit` (optional): Maximum number of events to return (default: `100`)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "query": "SrcIP = \"192.168.1.100\"",
  "from_date": "2026-03-01T00:00:00.000Z",
  "to_date": "2026-03-02T00:00:00.000Z",
  "event_type": "dns"
}
```

#### Returns

- Array of events of the specified type matching the query, along with pagination metadata

---

### 7. Get Alerts

Retrieve cloud detection alerts from SentinelOne with optional filters.

**Action:** `sentinel_one--get_alerts`

#### Parameters

- `limit` (optional): Maximum number of alerts to return (default: `50`)
- `created_after` (optional): Return alerts created after this ISO 8601 datetime
- `created_before` (optional): Return alerts created before this ISO 8601 datetime
- `severity` (optional): Filter by severity level (`critical`, `high`, `medium`, `low`)
- `rule_name` (optional): Filter alerts by rule name (partial match)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "severity": "critical",
  "created_after": "2026-03-01T00:00:00Z",
  "limit": 100
}
```

#### Returns

- Array of alert objects including rule name, severity, timestamps, and associated agent details
- Pagination metadata

---

## Best Practices

### Query Time Ranges

- Keep Deep Visibility query windows as narrow as possible — shorter ranges return faster and consume fewer resources
- Use ISO 8601 format with explicit UTC offset (`Z` or `+00:00`) to avoid timezone ambiguity

### Pagination

- `fetch_threats` automatically paginates through all results; avoid setting very large limits unnecessarily
- For Deep Visibility results, use the `limit` parameter to cap result size and reduce response time

### API Token Security

- Use a dedicated service account for the API token rather than a personal user account
- Restrict the token's scope to the minimum required permissions
- Rotate the token periodically following your organization's security policy

---

## FAQ

### What permissions does the API token need?

The token's permissions are determined by the SentinelOne user role. For this integration, the account should have read access to threats, events, and alerts. Deep Visibility queries require the Deep Visibility feature to be enabled on your account.

### Can I use multiple configurations?

Yes, use the `config_name` parameter to maintain multiple configurations (e.g., `production`, `staging`). Each configuration is stored separately.

### How long do Deep Visibility queries take?

Queries typically complete within a few seconds to a few minutes depending on the time range and scope. The integration polls every 20 seconds and will wait up to 10 minutes before timing out.

### What API version should I use?

The default is `v2.1`, which is the current stable version. Only change this if your SentinelOne instance requires a specific version.

### Why is my Deep Visibility query returning no results?

Ensure that the `from_date` and `to_date` range covers the expected event window, and that the PowerQuery syntax is correct. Deep Visibility data retention limits may also affect results for older time ranges.

---

**Related Integrations:**
- [IBM QRadar](ibm-qradar) - SIEM platform for security event analysis

**Back to:** [Integrations Overview](../)

**Last Updated:** March 2026
