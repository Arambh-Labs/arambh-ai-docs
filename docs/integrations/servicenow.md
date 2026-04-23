---
layout: default
title: ServiceNow
permalink: /docs/integrations-guide/servicenow/
parent: Integrations Guide
nav_order: 5
---

# ServiceNow

Integration with ServiceNow ITSM platform for incident management and service operations.

**Category:** Communications and Incident

**Description:** Search records, fetch and create incidents, update incident fields, and add comments to incidents in ServiceNow.

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [Advance Search](#1-advance-search)
  - [Fetch Incidents](#2-fetch-incidents)
  - [Create Incident](#3-create-incident)
  - [Update Incident](#4-update-incident)
  - [Add Comment to Incident](#5-add-comment-to-incident)
- [FAQ](#faq)
- [Troubleshooting](#troubleshooting)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Server URL** | String | Yes | ServiceNow instance URL (e.g., https://instance.service-now.com) |
| **Username** | String | Yes | ServiceNow username for Basic Authentication |
| **Password** | Password | Yes | ServiceNow password for Basic Authentication |

---

## Required Permissions

The user account must have the following ServiceNow roles:

- **itil** - Read and write access to incidents and related records
- **rest_service** - Access to the REST API

For read-only operations (Advance Search, Fetch Incidents), the `itil` role with read access is sufficient. For write operations (Create, Update, Add Comment), the account must have write access to the `incident` table.

---

## Setup Instructions

### 1. Create a Service Account in ServiceNow

- Navigate to **User Administration → Users**
- Click **New** to create a new user
- Set a username and password
- Assign the `itil` role (and `rest_service` if required by your instance)
- Save the user record

### 2. Configure in Arambh AI

- Settings → Integrations → ServiceNow
- Enter the Server URL (must be HTTPS)
- Enter the Username and Password
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Advance Search

Search records in any ServiceNow table using a `sysparm_query` filter expression.

**Action:** `service_now--advance_search`

#### Parameters

- `table_name` (required): The ServiceNow table to search (e.g., `incident`, `change_request`, `sys_user`)
- `adv_query` (required): A `sysparm_query` filter expression
  - Example: `active=true^priority=1`
  - Example: `sysparm_query=state=1^assigned_to=admin`
- `sysparm_limit` (optional): Maximum number of records to return (default: 10)
- `config_name` (optional): Configuration profile name

#### Common Query Expressions

- `active=true` - Only active records
- `priority=1` - High priority records
- `state=1` - New/open state
- `assigned_to=<username>` - Records assigned to a specific user
- `active=true^priority=1` - Combine filters with `^`

#### Returns

- Array of matching records from the specified table
- All field values as returned by the ServiceNow Table API

#### Example Response

```json
{
  "status": "success",
  "message": "Retrieved 3 record(s) from 'incident'",
  "data": [
    {
      "sys_id": "abc123def456",
      "number": "INC0010001",
      "short_description": "VPN connection failure",
      "state": "1",
      "priority": "1",
      "assigned_to": { "value": "user_sys_id", "display_value": "John Doe" }
    }
  ]
}
```

---

### 2. Fetch Incidents

Fetch incidents from the `incident` table with optional time range and query filters.

**Action:** `service_now--fetch_incidents`

#### Parameters

- `start_time` (optional): Start of the creation time window in ISO format (e.g., `2026-03-04T00:00:00.000Z`)
- `end_time` (optional): End of the creation time window in ISO format (e.g., `2026-03-04T23:59:59.999Z`)
- `adv_query` (optional): Additional `sysparm_query` filter appended to the time range filter (e.g., `priority=1^active=true`)
- `sysparm_limit` (optional): Maximum number of records to return (default: 10)
- `config_name` (optional): Configuration profile name

#### Time Template Support

Use time templates for relative time ranges:
- `{{now-1h}}` - One hour ago
- `{{now-24h}}` - 24 hours ago
- `{{now-7d}}` - 7 days ago

Templates are automatically converted to the ServiceNow datetime format.

#### Example

```json
{
  "start_time": "2026-03-04T00:00:00.000Z",
  "end_time": "2026-03-04T23:59:59.999Z",
  "adv_query": "priority=1^active=true",
  "sysparm_limit": 20
}
```

#### Returns

- Array of incident objects from the `incident` table
- All incident fields as returned by the ServiceNow Table API

---

### 3. Create Incident

Create a new incident in ServiceNow.

**Action:** `service_now--create_incident`

#### Parameters

- `short_description` (required): Brief summary of the incident
- `description` (optional): Detailed description of the incident
- `urgency` (optional): Urgency level — `1` (High), `2` (Medium), `3` (Low)
- `impact` (optional): Impact level — `1` (High), `2` (Medium), `3` (Low)
- `assignment_group` (optional): Name or sys_id of the group to assign the incident to
- `assigned_to` (optional): Username or sys_id of the user to assign the incident to
- `category` (optional): Category of the incident
- `caller_id` (optional): sys_id or username of the caller
- `additional_fields` (optional): Any extra fields to set as key-value pairs
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "short_description": "Database server unreachable",
  "description": "The primary database server has been unreachable since 14:00 UTC. Multiple services are affected.",
  "urgency": "1",
  "impact": "1",
  "assignment_group": "Database Admins",
  "category": "database"
}
```

#### Returns

- The newly created incident record including the assigned `number` (e.g., `INC0010042`) and `sys_id`

#### Example Response

```json
{
  "status": "success",
  "message": "Successfully created incident: INC0010042",
  "data": {
    "sys_id": "abc123def456",
    "number": "INC0010042",
    "short_description": "Database server unreachable",
    "state": "1",
    "urgency": "1",
    "impact": "1"
  }
}
```

---

### 4. Update Incident

Update fields on an existing ServiceNow incident by its `sys_id`.

**Action:** `service_now--update_incident`

#### Parameters

- `sys_id` (required): The `sys_id` of the incident to update
- `fields` (required): Dictionary of field names and values to update
- `config_name` (optional): Configuration profile name

#### Common Fields to Update

| Field | Values | Description |
|-------|--------|-------------|
| `state` | `1` (New), `2` (In Progress), `6` (Resolved), `7` (Closed) | Incident state |
| `priority` | `1`–`4` | Incident priority |
| `urgency` | `1` (High), `2` (Medium), `3` (Low) | Urgency level |
| `impact` | `1` (High), `2` (Medium), `3` (Low) | Impact level |
| `assigned_to` | Username or sys_id | Assignee |
| `assignment_group` | Group name or sys_id | Assignment group |
| `resolution_code` | `Solved (Work Around)`, `Solved (Permanently)`, etc. | Resolution code (required when resolving) |
| `resolution_notes` | String | Notes describing the resolution |

#### Example

```json
{
  "sys_id": "abc123def456",
  "fields": {
    "state": "6",
    "resolution_code": "Solved (Permanently)",
    "resolution_notes": "Root cause identified and patched. Monitoring in place."
  }
}
```

#### Returns

- The updated incident record

---

### 5. Add Comment to Incident

Add a work note or comment to an existing ServiceNow incident.

**Action:** `service_now--add_comment_to_incident`

#### Parameters

- `sys_id` (required): The `sys_id` of the incident to comment on
- `comment` (required): The comment text to add to the incident
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "sys_id": "abc123def456",
  "comment": "Escalated to the network team. Awaiting confirmation of fix deployment."
}
```

#### Returns

- The updated incident record confirming the comment was applied

#### Example Response

```json
{
  "status": "success",
  "message": "Successfully added comment to incident 'abc123def456'",
  "data": {
    "sys_id": "abc123def456",
    "number": "INC0010042"
  }
}
```

---


## FAQ

### Do I need an admin account to use this integration?

No. A service account with the `itil` role is sufficient for most operations. Admin access is only needed to create the service account itself.

### Can I search tables other than incidents?

Yes. Use the **Advance Search** action and specify any ServiceNow table name in the `table_name` parameter (e.g., `change_request`, `problem`, `sys_user`).

### How do I find the sys_id of an incident?

Use the **Fetch Incidents** or **Advance Search** action to retrieve incident records. Each record includes a `sys_id` field. You can also find it in the ServiceNow Console URL when viewing a record.

### Can I use multiple configurations?

Yes, use the `config_name` parameter to switch between configurations (e.g., "production", "staging"). Each configuration is stored separately.

### Why is the `caller_id` field blank on created incidents?

If `caller_id` is not provided, the field will be empty in the created record. Pass a valid ServiceNow username or `sys_id` to populate it.

---

## Troubleshooting

### HTTP 401 Unauthorized

- Verify the username and password are correct
- Ensure the user account is active in ServiceNow
- Confirm Basic Authentication is enabled on your ServiceNow instance

### HTTP 403 Forbidden

- The user account lacks the required role (`itil` or `rest_service`)
- Contact your ServiceNow administrator to assign the appropriate roles

### HTTP 404 Not Found

- Verify the Server URL is correct and includes the full instance URL (e.g., `https://instance.service-now.com`)
- Check that the table name in **Advance Search** exists in your ServiceNow instance

### Records Not Returned

- Check that your `sysparm_query` filter is correct — an invalid filter may return zero results silently
- Increase `sysparm_limit` if the result set is being truncated
- Verify the time range in **Fetch Incidents** matches when the records were created

---

**Related Integrations:**
- [Jira](jira) - Issue and project tracking
- [Microsoft Teams](microsoft-teams) - Team communications and notifications

**Back to:** [Integrations Overview](../)

**Last Updated:** April 2026
