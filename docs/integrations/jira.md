---
layout: default
title: Jira
permalink: /docs/integrations-guide/jira/
parent: Integrations Guide
nav_order: 10
---

# Jira

Create and manage issues, track projects, and automate ticket workflows using the Jira REST API v3.

**Category:** Communications and Incident

**Description:** Create tickets, update issue status, assign users, add comments, and list projects or issues using Basic Authentication with an API token.

---

## Table of Contents
- [Configuration](#configuration)
- [Required API Permissions](#required-api-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
- [FAQ](#faq)
- [Troubleshooting](#troubleshooting)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Server URL** | String | Yes | Jira instance base URL (e.g. `https://yourcompany.atlassian.net`) |
| **Email** | String | Yes | Email address used to log into Atlassian |
| **API Token** | Password | Yes | API token generated from your Atlassian account security settings |

---

## Required API Permissions

The following classic OAuth 2.0 scopes are required:

| Scope | Description |
|-------|-------------|
| `write:jira-work` | Create and edit issues, post comments, delete issues |
| `read:jira-work` | Read project and issue data, search for issues |
| `read:jira-user` | View user information — usernames, email addresses, avatars |

For Jira Cloud, generate an API token at:
```
https://id.atlassian.com/manage-profile/security/api-tokens
```

For project-level permissions (e.g. Delete Issues), ensure your account has the appropriate role in the target project. Navigate to your project → **Project settings → Permissions** to manage roles.

---

## Setup Instructions

### 1. Generate an API Token

- Log into your Atlassian account at `https://id.atlassian.com`
- Navigate to **Security → API tokens**
- Click **Create API token**
- Give it a label (e.g. "Arambh AI Integration")
- Copy the token — it is only shown once

### 2. Find Your Server URL

Your server URL is the base domain of your Jira instance:
- **Jira Cloud:** `https://yourcompany.atlassian.net`
- **Jira Server/DC:** `https://jira.yourcompany.com`

### 3. Find Your Project Key

The project key is visible in your Jira URL:
```
https://yourcompany.atlassian.net/jira/software/projects/SCRUM/list
```
In this example, the project key is **`SCRUM`**.

### 4. Configure in Arambh AI

- Navigate to Integrations → Jira
- In the **Configurations** tab, click **Add Configuration**
- Enter your **Server URL**, **Email**, and **API Token**
- Click **Save**
- In the **Actions** tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Create Ticket

Create a new issue in a Jira project.

**Action:** `jira--create_ticket`

#### Parameters

- `project_key` (required): The key of the Jira project (e.g. `SCRUM`)
- `summary` (required): Short title of the issue
- `issue_type` (optional): Issue type name — `Bug`, `Task`, `Story`, etc. Defaults to `Task`
- `description` (optional): Detailed description (plain text)
- `priority` (optional): Priority name — `High`, `Medium`, `Low`
- `assignee_account_id` (optional): Account ID of the user to assign the issue to
- `labels` (optional): List of label strings
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "project_key": "SCRUM",
  "summary": "Investigate suspicious login from 203.0.113.45",
  "issue_type": "Bug",
  "description": "Multiple failed login attempts detected from external IP.",
  "priority": "High",
  "assignee_account_id": "712020:ccfafa49-3087-436c-89a8-863bb1a59d72"
}
```

#### Returns

- Issue ID, key, and self URL

---

### 2. Get Ticket Details

Retrieve full details of a Jira issue by its ID or key.

**Action:** `jira--get_ticket`

#### Parameters

- `ticket_id` (required): Issue ID or key (e.g. `SCRUM-123`)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "ticket_id": "SCRUM-123"
}
```

#### Returns

- Full issue object including fields, status, assignee, comments, and attachments

---

### 3. List Projects

List all Jira projects accessible to the authenticated user.

**Action:** `jira--list_projects`

#### Parameters

- `config_name` (optional): Configuration profile name

#### Returns

- List of projects with `id`, `key`, `name`, and `projectTypeKey`

---

### 4. List Tickets

Search and list Jira issues using a JQL query. Automatically paginates via `nextPageToken` if Jira's internal page cap is lower than `max_results`.

**Action:** `jira--list_tickets`

#### Parameters

- `jql` (optional): JQL query string. Defaults to `status != Done ORDER BY created DESC`
- `max_results` (optional): Maximum total number of results to return. Defaults to `50`
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "jql": "project = SCRUM AND assignee = currentUser() AND status != Done",
  "max_results": 20
}
```

#### Returns

- `total`: Number of issues returned
- `issues`: List of issues, each with `id`, `key`, `summary`, `status`, `priority`, `issue_type`, `assignee`, `created`, `updated`

---

### 5. Search Users

Search for Jira users by name or email to retrieve their account ID.

**Action:** `jira--search_users`

#### Parameters

- `query` (optional): Search string — name or email address
- `max_results` (optional): Maximum number of users to return. Defaults to `50`
- `start_at` (optional): Pagination offset. Defaults to `0`
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "query": "gaurang@arambhlabs.com"
}
```

#### Returns

- List of users with `accountId`, `displayName`, `emailAddress`, and `active` status

**Note:** Requires **Browse users and groups** global permission. Without it, results will be empty.

---

### 6. Assign Issue to User

Assign a Jira issue to a specific user.

**Action:** `jira--assign_ticket`

#### Parameters

- `ticket_id` (required): Issue ID or key (e.g. `SCRUM-123`)
- `account_id` (required): Account ID of the user to assign. Pass empty string to unassign
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "ticket_id": "SCRUM-123",
  "account_id": "712020:ccfafa49-3087-436c-89a8-863bb1a59d72"
}
```

#### Returns

- Success confirmation message

---

### 7. Add Comment

Post a comment on a Jira issue.

**Action:** `jira--add_comment`

#### Parameters

- `ticket_id` (required): Issue ID or key (e.g. `SCRUM-123`)
- `comment` (required): Text content of the comment
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "ticket_id": "SCRUM-123",
  "comment": "Investigated the alert. The IP belongs to our VPN provider. Closing as false positive."
}
```

#### Returns

- Comment ID, creation timestamp, and author details

---

### 8. Get Comments

Retrieve comments on a Jira issue.

**Action:** `jira--get_comments`

#### Parameters

- `ticket_id` (required): Issue ID or key (e.g. `SCRUM-123`)
- `max_results` (optional): Maximum number of comments to return. Defaults to `50`
- `start_at` (optional): Pagination offset. Defaults to `0`
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "ticket_id": "SCRUM-123",
  "max_results": 10
}
```

#### Returns

- List of comments with body, author, and timestamps

---

### 9. Set Ticket Status

Transition a Jira issue to a new workflow status.

**Action:** `jira--set_ticket_status`

#### Parameters

- `ticket_id` (required): Issue ID or key (e.g. `SCRUM-123`)
- `status_name` (required): Target status name (e.g. `In Progress`, `Done`, `Closed`). Case-insensitive
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "ticket_id": "SCRUM-123",
  "status_name": "In Progress"
}
```

#### Returns

- Success confirmation message

**Note:** Available transitions depend on the project's workflow configuration. If the status is not available, the response will list valid transitions.

---

### 10. Update Ticket

Update one or more fields on an existing Jira issue.

**Action:** `jira--update_ticket`

#### Parameters

- `ticket_id` (required): Issue ID or key (e.g. `SCRUM-123`)
- `summary` (optional): New summary
- `description` (optional): New description (plain text)
- `priority` (optional): New priority name
- `assignee_account_id` (optional): Account ID of the new assignee
- `labels` (optional): Replacement list of labels
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "ticket_id": "SCRUM-123",
  "priority": "Critical",
  "labels": ["security", "incident"]
}
```

#### Returns

- Success confirmation message

---

### 11. Delete Ticket

Delete a Jira issue permanently.

**Action:** `jira--delete_ticket`

#### Parameters

- `ticket_id` (required): Issue ID or key (e.g. `SCRUM-123`)
- `delete_subtasks` (optional): If `true`, also deletes subtasks. Defaults to `false`
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "ticket_id": "SCRUM-123",
  "delete_subtasks": true
}
```

#### Returns

- Success confirmation message

**Note:** Requires the **Delete Issues** project permission for the authenticated user.

---

## FAQ

### Where do I find my account ID?

Your account ID is visible in your Jira profile URL or can be retrieved using `jira--search_users` with your email. It looks like: `712020:ccfafa49-3087-436c-89a8-863bb1a59d72`.

### Why does search_users return empty results?

The `search_users` action requires the **Browse users and groups** global permission in Jira. Without it, Jira silently returns an empty array. Ask your Jira admin to grant this permission to your account.

### Why can't I delete tickets?

Deleting issues requires the **Delete Issues** project permission. By default in Jira Cloud, only project Administrators have this permission. Go to your project's permission settings and ensure your role includes it.

### Can I use my Atlassian password instead of an API token?

No. Jira Cloud does not accept account passwords for API authentication — only API tokens are accepted, even for the same account.

### Where do I find the project key?

The project key is visible in the Jira URL after `/projects/`:
```
https://yourcompany.atlassian.net/jira/software/projects/SCRUM/list
                                                          ^^^^^ project key
```

### Can I use multiple configurations?

Yes, use the `config_name` parameter to maintain separate configurations for different Jira instances or accounts.

---

## Troubleshooting

### "You do not have permission to create issues in this project" (401)

**Solution:**
1. Ensure the **Email** field contains your full Atlassian email address
2. Verify the API token is correct — regenerate at `https://id.atlassian.com/manage-profile/security/api-tokens`
3. Test credentials: `curl -u "email@company.com:api-token" https://yourcompany.atlassian.net/rest/api/3/myself`
4. Check your role in the project has "Create Issues" permission

### "error.no-permission" (403) on search_users

**Solution:**
1. Your account lacks the **Browse users and groups** global permission
2. Ask a Jira admin to grant it at: **Jira Settings → System → Global Permissions**
3. Alternatively, use the account ID directly if already known (visible in your Jira profile URL)

### "The requested API has been removed" (410) on list_tickets

**Solution:**
The old `/rest/api/3/search` endpoint has been removed. The integration uses `/rest/api/3/search/jql` — ensure you are running the latest version of the integration.

### "Unbounded JQL queries are not allowed" (400)

**Solution:**
Always provide a JQL query with at least one filter condition. The integration defaults to `status != Done ORDER BY created DESC` when no JQL is provided.

### "nodename nor servname provided, or not known" (DNS error)

**Solution:**
1. Verify the `server_url` has no trailing spaces or extra characters
2. Test connectivity: `curl https://yourcompany.atlassian.net`
3. Check if a VPN is blocking external DNS resolution
4. Flush DNS cache: `sudo dscacheutil -flushcache && sudo killall -HUP mDNSResponder`

### Set status returns "Status not available"

**Solution:**
The response includes a list of valid transitions for the issue. Use one of those exact names. Transition availability depends on the issue's current status and the project's workflow.

---

**Related Integrations:**
- [Microsoft Teams](microsoft-teams) - Send notifications and approval cards

**Back to:** [Integrations Overview](../)

**Last Updated:** April 2026
