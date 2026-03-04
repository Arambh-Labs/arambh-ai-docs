---
layout: default
title: IBM QRadar
permalink: /docs/integrations-guide/ibm-qradar/
parent: Integrations Guide
nav_order: 4
---

# IBM QRadar

Integration with IBM QRadar SIEM platform for security event analysis and threat intelligence.

**Category:** SIEM (Security Information and Event Management)

**Description:** Query offenses, search events, execute AQL queries, and retrieve IP address information from IBM QRadar.

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [Get Offenses](#1-get-offenses)
  - [Get Events Related to Offense](#2-get-events-related-to-offense)
  - [Run Query](#3-run-query)
  - [Get Source IP](#4-get-source-ip)
  - [Get Destination IP](#5-get-destination-ip)
- [Best Practices](#best-practices)
- [Metadata](#metadata)
- [FAQ](#faq)
- [Troubleshooting](#troubleshooting)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Server URL** | String | Yes | QRadar server URL (e.g., https://qradar.company.com) |
| **API Token** | Password | Yes | QRadar authorized service token (SEC token) |

---

## Required Permissions

The API token must have the following QRadar permissions:

- **Offense Management** - Read offenses
- **Ariel** - Execute AQL queries and search events
- **Assets** - Read source and destination address information
- **System** - Health check endpoint access

**Token Type:** Authorized Service Token (SEC token)

### How to Generate SEC Token

1. Log into QRadar Console
2. Navigate to Admin → Authorized Services
3. Click "Create Authorized Service"
4. Select required capabilities
5. Copy the generated SEC token (shown only once)

---

## Setup Instructions

### 1. Create Authorized Service in QRadar

- QRadar Console → Admin → Authorized Services
- Click "New Authorized Service"
- Name: "Arambh AI Integration"
- Authentication: "Generate New Token"
- Security Profile: Select appropriate profile with required permissions
- Click "Create Service"
- **Important:** Copy the token immediately (shown only once)

### 2. Test API Access

Test your token with the following curl command:

```bash
curl -X GET "https://qradar.company.com/api/system/about" \
  -H "SEC: your-token-here" \
  -H "Accept: application/json"
```

### 3. Configure in Arambh AI

- Settings → Integrations → IBM QRadar
- Enter Server URL (must be HTTPS)
- Paste API Token
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Get Offenses

Retrieve security offenses from QRadar with optional filtering.

**Action:** `qradar--get_offenses`

#### Parameters

- `filter_string` (optional): QRadar API filter expression
  - Example: `status=OPEN`
  - Example: `severity>=7`
  - Example: `start_time>{{now-24h}}`
- `config_name` (optional): Configuration profile name

#### Common Filters

- `status=OPEN` - Only open offenses
- `status=CLOSED` - Only closed offenses
- `severity>=8` - High severity offenses (8-10)
- `magnitude>5` - Offenses with high magnitude
- `start_time>1709539200000` - After specific timestamp (epoch milliseconds)

#### Returns

- Array of offense objects with:
  - Offense ID, description, severity, magnitude
  - Status, start/close times
  - Source and destination IP addresses (resolved from IDs)
  - Categories, assigned user
  - Event and flow counts

#### Example Response

```json
{
  "status": "success",
  "message": "Retrieved 5 offense(s) from QRadar",
  "data": [
    {
      "id": 123,
      "description": "Multiple Login Failures Detected",
      "severity": 8,
      "magnitude": 6,
      "status": "OPEN",
      "source_address_ids": ["192.168.1.100", "10.0.0.50"],
      "local_destination_address_ids": ["172.16.0.10"],
      "start_time": 1709539200000,
      "event_count": 487
    }
  ]
}
```

**Note:** Source and destination IP addresses are automatically resolved from internal QRadar IDs.

---

### 2. Get Events Related to Offense

Retrieve security events associated with a specific offense.

**Action:** `qradar--get_events_related_to_offense`

#### Parameters

- `offense_id` (required): QRadar offense ID
- `start_time` (required): Start time in ISO format (e.g., `2026-03-04T00:00:00.000Z`)
- `end_time` (required): End time in ISO format (e.g., `2026-03-04T23:59:59.999Z`)
- `max_results` (optional): Maximum number of events to return (default: 10, avoid large numbers for performance)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "offense_id": "123",
  "start_time": "2026-03-04T00:00:00.000Z",
  "end_time": "2026-03-04T23:59:59.999Z",
  "max_results": 50
}
```

#### Returns

- Array of event objects with full event details
- Event properties: timestamp, source/destination IPs, ports, protocols
- Event names, categories, log sources
- Custom properties and payloads

#### Query Method

Uses Ariel Query Language (AQL) with `INOFFENSE()` function.

**Timeout:** 120 seconds (configurable via `ARIEL_TIMEOUT`)

---

### 3. Run Query

Execute a custom Ariel Query Language (AQL) search.

**Action:** `qradar--run_query`

#### Parameters

- `search_string` (required): AQL query to execute (minimum 3 characters)
- `config_name` (optional): Configuration profile name

#### Time Template Support

Use time templates for relative time ranges:
- `{{now-1h}}` - One hour ago
- `{{now-24h}}` - 24 hours ago
- `{{now-7d}}` - 7 days ago

Templates are automatically converted to epoch milliseconds.

#### Returns

- Query results based on SELECT statement
- Result metadata
- Event or flow objects matching query criteria

#### Polling

Query is submitted and polled every 20 seconds until complete or timeout (300s).

---

### 4. Get Source IP

Retrieve source IP address details by internal QRadar source address IDs.

**Action:** `qradar--get_source_ip`

#### Parameters

- `source_address_ids` (required): Comma-separated list of source address IDs
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "source_address_ids": "101,102,103"
}
```

#### Returns

- Array of source IP address objects
- IP addresses, network information
- First/last seen times
- Associated assets and domains

---

### 5. Get Destination IP

Retrieve local destination IP address details by internal QRadar destination address IDs.

**Action:** `qradar--get_destination_ip`

#### Parameters

- `destination_address_ids` (required): Comma-separated list of destination address IDs
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "destination_address_ids": "201,202,203"
}
```

#### Returns

- Array of local destination IP address objects
- IP addresses, network information
- Associated assets and services

---

## Best Practices

### 1. Query Performance

- **Limit Time Ranges:** Use specific time windows to reduce query scope
- **Use Specific Filters:** Add filters to narrow result sets
- **Avoid SELECT \*:** Specify only needed columns for large datasets
- **Use LIMIT Clause:** Add `LIMIT` to AQL queries to restrict result size

**Example:**
```sql
SELECT sourceip, destinationip, eventcount
FROM events
WHERE category = 'Authentication'
LAST 1 HOURS
LIMIT 100
```

### 4. Time Zones

- **UTC Format:** QRadar uses UTC internally
- **Convert Local Times:** Convert local times to UTC/ISO format
- **Use Epoch Milliseconds:** Use epoch milliseconds for precision
- **Consistent Format:** Maintain consistent time format across queries

**Example Time Conversions:**
```python
# ISO format
start_time = "2026-03-04T00:00:00.000Z"

# Epoch milliseconds
start_time = 1709539200000

# Relative time template
start_time = "{{now-24h}}"
```

---

## FAQ

### Do I need special permissions to create an authorized service?

Yes, you need administrator access in QRadar to create authorized services and generate SEC tokens.

### Can I use multiple configurations?

Yes, use the `config_name` parameter to maintain multiple configurations (e.g., "production", "staging"). Each configuration is stored separately.

### How do I find offense IDs?

Use the "Get Offenses" action with appropriate filters, or check the QRadar Console. Offense IDs are visible in the offense list and offense details pages.

### Can I query historical data?

Yes. Use specific time ranges and check QRadar's data retention policy for your deployment.

### Why are some IPs showing as numbers?

This happens when QRadar internal IDs cannot be resolved to IP addresses. The IDs may be historical or no longer in the active database.

### What is the difference between events and flows?

- **Events:** Individual security events from log sources (e.g., firewall logs, authentication logs)
- **Flows:** Network flow data showing network connections and traffic volume

Use `FROM events` for event-based queries and `FROM flows` for network traffic analysis.

### Does the integration support custom properties?

Yes, custom properties are included in event results. Reference them by name in your AQL queries.

### How often should I rotate my SEC token?

Follow your organization's security policy. Generally, rotate tokens every 90-180 days or whenever there's a security concern.

---

**Related Integrations:**
- [Splunk](splunk) - Alternative SIEM platform
- [Microsoft Azure Sentinel](azure-sentinel) - Cloud-native SIEM

**Back to:** [Integrations Overview](../)

**Last Updated:** March 2026
