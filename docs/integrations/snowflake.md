---
layout: default
title: Snowflake
permalink: /docs/integrations-guide/snowflake/
parent: Integrations Guide
nav_order: 12
---

# Snowflake

Run SQL queries and retrieve alert definitions and execution history from Snowflake using the SQL REST API and the Snowflake REST Alert API.

**Category:** SIEM, Database

**Description:** Execute SQL queries against any Snowflake database, list alert definitions, and retrieve alert execution history using Programmatic Access Token (PAT) authentication.

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
- [FAQ](#faq)
- [Troubleshooting](#troubleshooting)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Account Identifier** | String | Yes | Snowflake account identifier (e.g. `myorg-myaccount` or `xy12345.us-east-1.aws`) |
| **Programmatic Access Token** | Password | Yes | PAT generated from Snowflake for API authentication |
| **Role** | String | No | Snowflake role to assume for all requests (e.g. `SYSADMIN`) |

---

## Required Permissions

The Snowflake user associated with the PAT must have privileges to:

| Privilege | Required For |
|-----------|-------------|
| `USAGE` on database and schema | Running queries and fetching alerts |
| `SELECT` on target tables/views | `run_query` |
| `MONITOR` on alerts | `fetch_alerts`, `fetch_alert_history` |
| `IMPORTED PRIVILEGES` on `SNOWFLAKE` DB | `fetch_alert_history` with `use_account_usage: true` |

---

## Setup Instructions

### 1. Find Your Account Identifier

**Preferred format:** `<org>-<account_name>` (e.g. `myorg-myaccount`)

- Log into Snowflake → **Admin → Accounts** → hover over your account name to see the full identifier
- Or run in a Snowflake worksheet:
```sql
SELECT CURRENT_ORGANIZATION_NAME() || '-' || CURRENT_ACCOUNT_NAME();
```

**Legacy format:** `<locator>.<region>.<cloud>` (e.g. `xy12345.us-east-1.aws`)
- Visible in your Snowflake login URL: `https://xy12345.snowflakecomputing.com`

### 2. Generate a Programmatic Access Token (PAT)

**Via Snowsight UI:**
1. Log into Snowflake → **Admin → Security → Programmatic Access Tokens**
2. Click **+ Programmatic Access Token**
3. Enter a name, set expiry (1–365 days), optionally restrict to a role
4. Copy the token secret immediately — it is only shown once

**Via SQL:**
```sql
ALTER USER <your_username> ADD PROGRAMMATIC ACCESS TOKEN my_pat
  DAYS_TO_EXPIRY = 90;
```

### 3. Configure Network Policy

Snowflake PATs require a network policy to be attached to the user before API access is allowed.

**Via Snowsight UI:**
1. Go to **Admin → Security → Network Policies** → **+ Network Policy**
2. Add your server IP under **Allowed IP Addresses** (use `0.0.0.0/0` for testing)
3. Save, then go to **Admin → Users & Roles → Users** → select your user → set **Network Policy**

**Via SQL:**
```sql
CREATE NETWORK POLICY allow_api_access
  ALLOWED_IP_LIST = ('0.0.0.0/0');

ALTER USER <your_username> SET NETWORK_POLICY = allow_api_access;
```

### 4. Configure in Arambh AI

- Navigate to **Integrations → Snowflake**
- In the **Configurations** tab, click **Add Configuration**
- Enter your **Account Identifier**, **Programmatic Access Token**, and optionally a **Role**
- Click **Save**
- In the **Actions** tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Run Query

Execute a SQL statement against Snowflake and return the results.

**Action:** `snowflake--run_query`

#### Parameters

- `query` (required): SQL statement to execute. Supports `SELECT`, `SHOW`, `CALL`, and most read-oriented statements
- `database` (optional): Override the default database for this query
- `schema` (optional): Override the default schema for this query
- `warehouse` (optional): Override the warehouse to use for this query
- `role` (optional): Override the Snowflake role for this query
- `max_results` (optional): Maximum rows to return. Defaults to `100`. A `LIMIT` clause in the query takes precedence
- `timeout` (optional): Query timeout in seconds. Defaults to `120`
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "query": "SELECT * FROM my_db.public.login_events WHERE status = 'FAILED' AND event_time >= DATEADD(hour, -24, CURRENT_TIMESTAMP()) LIMIT 100"
}
```

#### Returns

- `data`: List of rows as JSON objects
- `message`: Row count summary

#### Notes

- For `SELECT` statements without a `LIMIT` clause, the integration automatically appends `LIMIT <max_results>`
- Long-running queries are polled automatically (up to 2 minutes)
- Large result sets are fetched across partitions and returned as a single flat list

---

### 2. Fetch Alerts

List Snowflake alert definitions in a given database and schema using the Snowflake REST Alert API.

**Action:** `snowflake--fetch_alerts`

**API endpoint used:** `GET /api/v2/databases/{database}/schemas/{schema}/alerts`

#### Parameters

- `database` (required): Database to list alerts from (e.g. `MY_DB`)
- `schema` (required): Schema to list alerts from (e.g. `PUBLIC`)
- `like` (optional): Filter alert names by SQL LIKE pattern (e.g. `SECURITY_%`)
- `limit` (optional): Maximum number of alerts to return
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "database": "MY_DB",
  "schema": "PUBLIC"
}
```

#### Returns

- List of alert definition objects including name, state, condition, schedule, and owner

#### Notes

- Returns alert **definitions** (configuration), not execution history. Use `snowflake--fetch_alert_history` for execution records
- The user must have `MONITOR` privilege on the alerts or `OWNERSHIP` of them to see results

---

### 3. Fetch Alert History

Retrieve Snowflake alert execution history — when alerts ran, their state, and error details.

**Action:** `snowflake--fetch_alert_history`

#### Parameters

- `alert_name` (optional): Filter results to a specific alert name
- `database` (optional): Database where the alert is defined (e.g. `SNOWFLAKE_LEARNING_DB`). Required when `use_account_usage` is `false`, since `INFORMATION_SCHEMA.ALERT_HISTORY()` must be qualified with a database prefix
- `scheduled_time_from` (optional): Start of the time range. Accepts Krafter AI time templates (e.g. `{{now - 1h}}`, `{{now - 7d}}`) or a plain ISO-8601 string (e.g. `2026-07-01T00:00:00Z`). Defaults to 24 hours ago
- `scheduled_time_to` (optional): End of the time range. Accepts Krafter AI time templates (e.g. `{{now}}`) or a plain ISO-8601 string (e.g. `2026-07-13T23:59:59Z`). Defaults to now
- `state` (optional): Filter by execution state — `SUCCEEDED`, `FAILED`, `CANCELLED`, `SKIPPED`
- `use_account_usage` (optional): When `true`, queries `SNOWFLAKE.ACCOUNT_USAGE.ALERT_HISTORY` (up to 365 days). When `false` (default), queries `{database}.INFORMATION_SCHEMA.ALERT_HISTORY()` (last 7 days)
- `max_results` (optional): Maximum rows to return. Defaults to `100`
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "alert_name": "MY_SECURITY_ALERT",
  "database": "MY_DB",
  "scheduled_time_from": "{{now - 24h}}",
  "state": "FAILED"
}
```

#### Returns

- `data`: List of execution records with `name`, `state`, `condition_query_id`, `action_query_id`, `scheduled_time`, `completed_time`, `error_message`

#### Notes

- `INFORMATION_SCHEMA.ALERT_HISTORY()` covers the last 7 days only
- For history beyond 7 days, set `use_account_usage: true` — requires `IMPORTED PRIVILEGES` on the `SNOWFLAKE` database
- Equivalent SQL (as used by the integration):
```sql
SELECT name, state, condition_query_id, action_query_id,
       scheduled_time, completed_time, error_message
FROM TABLE(INFORMATION_SCHEMA.ALERT_HISTORY(
  SCHEDULED_TIME_RANGE_START => '<from>'::TIMESTAMP_LTZ,
  ALERT_NAME => '<alert_name>'))
ORDER BY scheduled_time DESC
LIMIT 100;
```

---

## FAQ

### Where do I find my account identifier?

Go to **Admin → Accounts** in Snowflake, hover over your account name, and the full `org-account` identifier is shown. Alternatively run:
```sql
SELECT CURRENT_ORGANIZATION_NAME() || '-' || CURRENT_ACCOUNT_NAME();
```

### Why is the PAT token only shown once?

Snowflake does not store the token secret after creation. Copy it immediately when generated. If lost, revoke the token and create a new one (max 15 tokens per user).

### Can I use multiple configurations?

Yes, use the `config_name` parameter to maintain separate configurations for different Snowflake accounts or roles.

### What is the difference between `fetch_alerts` and `fetch_alert_history`?

- `fetch_alerts` returns alert **definitions** — what alerts exist, their condition SQL, schedule, and current enabled/disabled state
- `fetch_alert_history` returns alert **execution records** — when each alert ran, whether it succeeded or failed, and any error messages

---

## Troubleshooting

### "Network policy is required" (401, code 390432)

**Solution:**
Snowflake PATs require a network policy to be attached to the user. Create one and assign it:
```sql
CREATE NETWORK POLICY allow_api_access ALLOWED_IP_LIST = ('0.0.0.0/0');
ALTER USER <your_username> SET NETWORK_POLICY = allow_api_access;
```
Or configure via **Admin → Security → Network Policies** in Snowsight.

### "Schema does not exist or not authorized" (002003)

**Solution:**
1. Verify the database and schema names are correct and match the output of `SHOW SCHEMAS IN DATABASE <db>`
2. Ensure the user's role has `USAGE` privilege on the database and schema
3. Note that schema names are case-sensitive in the REST API

### "pat_token is required" error

**Solution:**
The Programmatic Access Token field is empty in your configuration. Generate a PAT in Snowflake under **Admin → Security → Programmatic Access Tokens** and update the configuration.

### Query times out

**Solution:**
1. Increase the `timeout` parameter (default 120 seconds)
2. Add a `LIMIT` clause to reduce result set size
3. Ensure the warehouse is running — a suspended warehouse adds resume time

---

**Related Integrations:**
- [IBM QRadar](ibm-qradar) — SIEM log querying
- [Google Chronicle SecOps](google-chronicle-secops) — Security log querying

**Back to:** [Integrations Overview](../)

**Last Updated:** July 2026
