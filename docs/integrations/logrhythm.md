---
layout: default
title: LogRhythm SIEM
permalink: /docs/integrations-guide/logrhythm/
parent: Integrations Guide
nav_order: 7
---

# LogRhythm SIEM

Integration with the LogRhythm SIEM platform for alarm management and log/event search.

**Category:** SIEM (Security Information and Event Management)

**Description:** Retrieve alarms, fetch events associated with an alarm, and run searches against LogRhythm's Search API (Data Indexer).

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [Get Alarms](#1-get-alarms)
  - [Get Alarm Events](#2-get-alarm-events)
  - [Run Query](#3-run-query)
- [Best Practices](#best-practices)
- [FAQ](#faq)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Server URL** | String | Yes | Base URL of the LogRhythm server (e.g. `https://logrhythm.example.com:8501`). Endpoint paths like `/lr-alarm-api` and `/lr-search-api` are appended to this URL. |
| **API Token** | Password | Yes | LogRhythm API JSON Web Token (JWT). It is sent as-is in the `Authorization` header, **without** a `Bearer` prefix. |

---

## Required Permissions

The API token must be authorized for the following LogRhythm APIs:

- **Alarm API** (`lr-alarm-api`) - Read alarms and alarm events
- **Search API** (`lr-search-api`) - Execute searches against the Data Indexer

**Token Type:** LogRhythm API JWT

---

## Setup Instructions

### 1. Generate an API Token

1. Log into the LogRhythm Web Console
2. Navigate to **Deployment Manager** → **Users** (or your organization's API access management page)
3. Select the service account you want to use for the integration (or create a new one with least-privilege access)
4. Generate an API JSON Web Token (JWT) for that account
5. Copy the generated token — treat it as a secret and store it securely

### 2. Configure in Arambh AI

- Navigate to Integrations → LogRhythm SIEM
- In the 'Configurations' tab, click on 'Add Configuration'
- Enter the Server URL (include the port if your deployment uses a non-default one, e.g. `:8501`)
- Paste the API Token
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Get Alarms

Retrieve alarms from LogRhythm with optional filtering.

**Action:** `logrhythm--get_alarms`

#### Parameters

- `alarm_status` (optional): Filter by alarm status — one of `New`, `Opened`, `Working`, `Escalated`, `Closed`, `Closed_FalseAlarm`, `Closed_Resolved`, `Closed_Unresolved`, `Closed_Reported`, `Closed_Monitor`
- `alarm_rule_name` (optional): Filter by the name of the alarm rule that generated the alarm
- `entity_name` (optional): Filter by entity name
- `date_inserted` (optional): Only return alarms inserted on or after this time (ISO 8601 timestamp or time template, e.g. `{{now - 24h}}`)
- `notification` (optional): Filter by whether the requesting user is on the alarm's notification list
- `case_association` (optional): Filter by associated case
- `offset` (optional): Number of alarms to skip before collecting results (default: `0`)
- `count` (optional): Maximum number of alarms to return (default: `50`)
- `orderby` (optional): Field to sort results by — one of `AlarmRuleName`, `AlarmStatus`, `DateInserted`, `entityName` (default: `DateInserted`)
- `dir` (optional): Sort direction — `ascending` or `descending` (default: `ascending`)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "alarm_status": "New",
  "date_inserted": "{{now - 24h}}",
  "count": 25
}
```

#### Returns

- Array of alarm objects including alarm ID, rule name, status, entity, and insertion date

---

### 2. Get Alarm Events

Fetch events associated with a specific LogRhythm alarm.

**Action:** `logrhythm--get_alarm_events`

#### Parameters

- `alarm_id` (required): The numeric ID of the alarm to retrieve events for
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "alarm_id": "123456"
}
```

#### Returns

- Event data associated with the specified alarm

---

### 3. Run Query

Run a log/event search against LogRhythm's Search API (Data Indexer).

**Action:** `logrhythm--run_query`

#### Parameters

- `start_time` (required): Start time for the search in ISO 8601 format or a time template (e.g. `{{now - 1h}}`)
- `end_time` (required): End time for the search in ISO 8601 format or a time template (e.g. `{{now}}`)
- `query_filter` (optional): Raw LogRhythm `queryFilter` object used to narrow the search (`msgFilterType`, `filterGroup` with nested `filterItems`), following the LogRhythm Search API filter schema (`filterType`/`valueType` enums for fields like IP, Host, User, Login, Message, etc). Omit to return all logs/events in the time range.
- `max_results` (optional): Maximum number of results to return (default: `100`)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "start_time": "{{now - 1h}}",
  "end_time": "{{now}}",
  "query_filter": {
    "msgFilterType": 1,
    "filterGroup": {
      "filterItemType": "Group",
      "fieldOperator": "Or",
      "filterMode": "Simple",
      "filterItems": [
        {
          "filterType": "User",
          "fieldOperator": "Contains",
          "valueType": "String",
          "value": { "value": "jdoe" }
        }
      ]
    }
  },
  "max_results": 200
}
```

#### Returns

- Search result data matching the query criteria

#### Query Method

The action creates a search task (`/lr-search-api/actions/search-task`) and then immediately fetches results (`/lr-search-api/actions/search-result`) using the returned task ID. There is no polling/wait step, so very large or complex searches may return partial or empty results if the search has not finished on the LogRhythm side — narrow the time range and filters for best results.

---

## Best Practices

### Query Performance

- **Limit Time Ranges:** Use specific, narrow time windows to keep searches fast and reduce the chance of incomplete results
- **Use `query_filter`:** Narrow results server-side with a `queryFilter` instead of requesting broad, unfiltered ranges
- **Cap `max_results`:** Keep `max_results` reasonable for the query — large values increase search and response time

### Time Handling

- Use time templates (e.g. `{{now - 1h}}`, `{{now - 24h}}`) for relative time ranges; they are converted automatically
- Use explicit ISO 8601 timestamps when you need a precise, fixed window

---

## FAQ

### Can I use multiple configurations?

Yes, use the `config_name` parameter to maintain multiple configurations (e.g., `production`, `staging`). Each configuration is stored separately.

### How do I find an alarm ID?

Use the "Get Alarms" action to list alarms — each alarm object includes its ID, which can then be passed to "Get Alarm Events".

---

**Related Integrations:**
- [IBM QRadar](ibm-qradar) - Alternative SIEM platform
- [Google Chronicle SecOps](google-chronicle-secops) - Cloud-native SIEM

**Back to:** [Integrations Overview](../)

**Last Updated:** July 2026
