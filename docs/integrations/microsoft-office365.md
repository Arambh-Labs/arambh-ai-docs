---
layout: default
title: Microsoft Office 365
permalink: /docs/integrations-guide/microsoft-office365/
parent: Integrations Guide
nav_order: 2
---

# Microsoft Office 365

Send and manage emails using Office 365 through Microsoft Graph API.

**Category:** Communication

**Description:** Send emails with attachments, search mailboxes, and reply to emails using your Office 365 account.

---

## Table of Contents
- [Configuration](#configuration)
- [Required API Permissions](#required-api-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
- [Metadata](#metadata)
- [FAQ](#faq)
- [Troubleshooting](#troubleshooting)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Tenant ID** | String | Yes | Azure AD Directory (tenant) ID |
| **Client ID** | String | Yes | Application (client) ID from Azure AD |
| **Client Secret** | Password | Yes | Client secret value from Azure AD |
| **User ID** | String | Yes | Mailbox email address to access (e.g., service-account@domain.com) |
| **Server URL** | String | No | Microsoft Graph API endpoint (default: https://graph.microsoft.com/) |

---

## Required API Permissions

Your Azure AD application requires the following Microsoft Graph API permissions:

- `Mail.Send` - Send mail as any user
- `Mail.ReadWrite` - Read and write mail in all mailboxes
- `User.Read.All` - Read all users' basic profiles

**Permission Type:** Application permissions (service-to-service)

**OAuth Scope:** `https://graph.microsoft.com/.default`

---

## Setup Instructions

### 1. Register Application in Azure AD

- Follow the same registration process as Azure AD integration
- Go to Azure Portal → Azure Active Directory → App registrations
- Click "New registration"
- Use a name like "Arambh AI - Office 365 Integration"
- Select "Accounts in this organizational directory only"
- Click "Register"

### 2. Configure API Permissions

- Navigate to "API permissions"
- Click "Add a permission" → Microsoft Graph → Application permissions
- Add the following permissions:
  - `Mail.Send`
  - `Mail.ReadWrite`
  - `User.Read.All`
- Click "Grant admin consent" (requires admin privileges)

### 3. Create Client Secret

- Navigate to "Certificates & secrets"
- Click "New client secret"
- Provide a description and select expiration period
- Copy the secret value (only shown once)

### 4. Identify Service Account

- Use a dedicated service mailbox or shared mailbox
- Note the full email address (e.g., `arambh-service@company.com`)
- Ensure the mailbox is active and accessible

### 5. Configure in Arambh AI

- Navigate to Integrations → Microsoft Office365
- In the 'Configurations' tab, click on 'Add Configuration'
- Enter Tenant ID (from Overview page)
- Enter Client ID (Application ID from Overview page)
- Enter Client Secret (copied in previous step)
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Send Email

Send emails with support for attachments, templates, and multiple recipients.

**Action:** `microsoft_office365--send_email`

#### Parameters

- `subject` (required): Email subject line
- `body` (required): HTML formatted body, plain text, template path, or .html file path
- `to_recipients` (required): Array of recipient email addresses
- `cc_recipients` (optional): Array of CC recipient email addresses
- `bcc_recipients` (optional): Array of BCC recipient email addresses
- `attachments` (optional): Array of attachment objects:
  - `name`: Filename with extension
  - `file_path`: Path to the file to attach
- `template_name` (optional): Name of email template to use
- `template_data` (optional): JSON object with template variable values
- `poll_for_sent_email` (optional): Set to `true` if waiting for reply in workflow
- `config_name` (optional): Configuration profile name

#### Example Request

```json
{
  "subject": "Security Alert: Suspicious Activity Detected",
  "body": "<h1>Alert</h1><p>We detected suspicious login attempts from IP 192.168.1.100</p>",
  "to_recipients": ["security-team@company.com"],
  "cc_recipients": ["soc-manager@company.com"],
  "attachments": [
    {
      "name": "security-report.pdf",
      "file_path": "/tmp/reports/security-report.pdf"
    }
  ]
}
```

#### Returns

- Success confirmation with message ID
- Tracking information if polling enabled

---

### 2. Search Emails

Search mailbox with structured filters and natural language time expressions.

**Action:** `microsoft_office365--search_emails`

#### Parameters

- `folder` (optional): Mail folder to search (default: "inbox")
- `limit` (optional): Maximum emails to return (1-100, default: 10)
- `unread_only` (optional): Return only unread emails (default: false)
- `receivedDateTime` (optional): Time filter using:
  - Natural language: `{{now - 1d}}` (yesterday), `{{now - 7d}}` (last week)
  - ISO dates: `2026-03-04`
  - Date ranges: `2026-03-01:2026-03-04`
  - Comparisons: `>2026-03-01` (after date), `<2026-03-04` (before date)
- `sender` (optional): Filter by sender name or email (no "from:" prefix needed)
- `subject` (optional): Filter by subject line
- `markEmailsAsRead` (optional): Mark retrieved emails as read (default: false)
- `config_name` (optional): Configuration profile name

#### Example Search Queries

```json
{
  "folder": "inbox",
  "limit": 20,
  "unread_only": true,
  "receivedDateTime": "{{now - 3d}}",
  "sender": "security@company.com",
  "subject": "alert"
}
```

#### Returns

- Array of email objects with metadata
- Body preview and full body
- Attachment information
- Conversation ID for threading

---

### 3. Reply to Email

Reply to a specific email with optional attachments.

**Action:** `microsoft_office365--reply_to_email`

#### Parameters

- `message_id` (required): ID of the email to reply to
- `comment` (required): HTML formatted reply message content
- `to_recipients` (optional): Override default reply-to recipients
- `attachments` (optional): Array of attachment objects:
  - `name`: Filename with extension
  - `content`: Base64 encoded file content
- `config_name` (optional): Configuration profile name

#### Example Request

```json
{
  "message_id": "AAMkAGI2TG93AAA=",
  "comment": "<p>Thank you for your inquiry. The issue has been resolved.</p>",
  "attachments": [
    {
      "name": "resolution-report.pdf",
      "content": "JVBERi0xLjQKJeLjz9MK..."
    }
  ]
}
```

#### Returns

- Success confirmation (HTTP 202 Accepted)
- Reply sent status

---

**Related Integrations:**
- [Microsoft Azure Active Directory](microsoft-azure-ad) - User management
- [Microsoft Teams](microsoft-teams) - Teams messaging

**Back to:** [Integrations Overview](../)

**Last Updated:** March 2026
