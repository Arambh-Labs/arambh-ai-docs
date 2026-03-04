---
layout: default
title: Microsoft Teams
permalink: /docs/integrations-guide/microsoft-teams/
parent: Integrations Guide
nav_order: 3
---

# Microsoft Teams

Send messages and manage communication in Microsoft Teams channels and chats.

**Category:** Communication

**Description:** Post messages to channels, reply to conversations, send direct messages, and send approval cards using Microsoft Graph API or Bot Framework.

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
| **Team ID** | String | Yes | Microsoft Teams Team ID where channels are located |
| **Authentication Type** | Select | Yes | Choose: "Delegated Permissions" or "Bot Framework" |
| **Graph API URL** | String | No* | Microsoft Graph API endpoint (default: https://graph.microsoft.com) |
| **Redirect URI** | String | Yes* | Redirect URI for OAuth (required for delegated permissions) |
| **Service URL** | String | Yes** | Bot Framework Service URL (required for bot authentication) |
| **Authorization Code** | Password | Yes* | OAuth authorization code (for initial setup of delegated permissions) |

\* Required only for Delegated Permissions
\*\* Required only for Bot Framework

---

## Required API Permissions

### For Delegated Permissions

Your Azure AD application requires the following Microsoft Graph API permissions:

- `ChannelMessage.Send` - Send messages to channels
- `Channel.ReadBasic.All` - Read basic channel properties
- `Chat.ReadWrite` - Read and write chats
- `User.Read.All` - Read user profiles

**Permission Type:** Delegated permissions (on behalf of a user)

**OAuth Scope:** `https://graph.microsoft.com/.default`

### For Bot Framework

- Bot must be registered in Azure Bot Service
- Bot must be added to the target Team
- Requires Bot Framework scope: `https://api.botframework.com/.default`

**Permission Type:** Application permissions (service-to-service)

---

## Setup Instructions

### Option A: Delegated Permissions Setup

#### 1. Register Application

- Azure Portal → App registrations → New registration
- Name: "Arambh AI - Teams Integration"
- Redirect URI: `http://localhost:8000` (or your callback URL)
- Click "Register"

#### 2. Configure API Permissions

- Navigate to "API permissions"
- Click "Add a permission" → Microsoft Graph → Delegated permissions
- Add the following permissions:
  - `ChannelMessage.Send`
  - `Channel.ReadBasic.All`
  - `Chat.ReadWrite`
  - `User.Read.All`
- Click "Grant admin consent" (requires admin privileges)

#### 3. Get Authorization Code

Navigate to the following URL in your browser (replace `{tenant-id}` and `{client-id}` with your values):

```
https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/authorize?client_id={client-id}&response_type=code&redirect_uri=http://localhost:8000&scope=https://graph.microsoft.com/.default
```

Copy the authorization code from the redirect URL after authentication.

#### 4. Find Team ID

- Open Microsoft Teams
- Click "..." next to your team name
- Select "Get link to team"
- Extract Team ID from URL: the value after `groupId=`

#### 5. Configure in Arambh AI

- Navigate to Integrations → Microsoft Teams
- In the 'Configurations' tab, click on 'Add Configuration'
- Select Authentication Type: "Delegated Permissions"
- Enter Tenant ID (from Overview page)
- Enter Client ID (Application ID from Overview page)
- Enter Client Secret (copied earlier)
- Enter Team ID
- Enter Redirect URI (must match Azure AD configuration)
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

### Option B: Bot Framework Setup

#### 1. Register Bot in Azure

- Azure Portal → Create Resource → "Azure Bot"
- Create bot with messaging endpoint
- Configure bot settings
- Note the Service URL provided

#### 2. Configure Bot Credentials

- Use same Client ID and Secret as app registration
- Ensure app has appropriate Bot Framework permissions

#### 3. Add Bot to Team

- Microsoft Teams → Apps → Upload custom app
- Create app manifest for your bot
- Install bot to target team
- Verify bot appears in team members

#### 4. Get Service URL

- Service URL is typically in format: `https://smba.trafficmanager.net/amer/{region-id}/`
- Found in bot configuration in Azure Portal
- Contact Azure support if Service URL is unclear

#### 5. Configure in Arambh AI

- Navigate to Settings → Integrations → Microsoft Teams
- Select Authentication Type: "Bot Framework"
- Enter Tenant ID, Client ID, Client Secret
- Enter Team ID
- Enter Service URL from bot configuration
- Click "Connect" to test connection

---

## Supported Actions

### 1. Send Channel Message

Post a new message to a Microsoft Teams channel.

**Action:** `microsoft_teams--send_channel_message`

#### Parameters

- `channel_name` (required): Name of the Teams channel (e.g., "General", "Security Alerts")
- `message` (required): Message content (supports HTML formatting)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "channel_name": "Security Alerts",
  "message": "<h2>⚠️ Security Alert</h2><p>Suspicious login detected from <b>192.168.1.100</b></p><p>Time: 2026-03-04 10:30 AM</p>"
}
```

#### Returns

- Message ID
- Creation timestamp
- Message web URL

---

### 2. Reply to Channel Message

Reply to an existing message in a Teams channel (creates a thread).

**Action:** `microsoft_teams--reply_channel_message`

#### Parameters

- `channel_name` (required): Name of the channel
- `reply_message` (required): Reply content (text or HTML)
- `message_id` (optional): ID of message to reply to (if not provided, replies to latest message)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "channel_name": "Security Alerts",
  "message_id": "1234567890",
  "reply_message": "Investigated. The login was from our VPN. Marking as resolved."
}
```

#### Returns

- Reply message ID
- Thread information
- Success status

**Note:** If `message_id` is not provided, the action automatically finds the latest root message in the channel and replies to it.

---

### 3. Send Chat Message

Send a direct message to a user via 1-on-1 chat.

**Action:** `microsoft_teams--send_chat_message`

#### Parameters

- `user_email` (required): Email address of the user to message
- `message` (required): Message content
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "user_email": "john.doe@company.com",
  "message": "Hi John, please review the security incident assigned to you."
}
```

#### Returns

- Message ID
- Chat ID
- Delivery status

**Note:** If a 1-on-1 chat doesn't exist with the user, it will be created automatically.

---

### 4. Send Approval Card (Bot Framework Only)

Send an interactive approval card for workflow approvals.

**Action:** `microsoft_teams--send_ask_approval_card`

#### Parameters

- `conversation_id` (required): Bot conversation ID
- `investigation_id` (required): Investigation/case identifier
- `investigation_name` (required): Name of the investigation
- `workplan_execution_id` (required): Workflow execution identifier
- `information_about_investigation` (required): Formatted summary with:
  - **Action Required:** What needs approval (bold)
  - **Summary:** Investigation details (bold heading)
  - **Next Steps after approval:** What happens if approved (bold heading)
- `config_name` (optional): Configuration profile name

#### Card Features

- Interactive approve/decline buttons
- Reason/justification text field
- Direct link to investigation details
- Formatted with markdown support

#### Returns

- Message ID
- Investigation tracking data
- Card activity ID

#### Example Summary Format

```
**Action Required: Approve blocking of malicious IP**

**Summary:** Investigation #1234 detected brute force login attempts from IP 203.0.113.45 targeting 15 user accounts. The attack started at 08:00 AM and continued for 2 hours with 487 failed login attempts.

**Next Steps after approval:** The system will automatically add the IP to the firewall block list, terminate any active sessions from this IP, and notify affected users to change their passwords.
```

**Note:** This action is only available when using Bot Framework authentication.

---

## FAQ

### Do I need admin consent for this integration?

Yes, Microsoft Teams integration requires admin consent in Azure Portal to grant application permissions. A Global Administrator or Application Administrator must approve the permissions.

### Can Teams Bot Framework and Delegated Permissions be used together?

No, choose one authentication method per configuration. Bot Framework is recommended for automated workflows, while Delegated Permissions work better for user-delegated scenarios.

### How do I find my Microsoft Teams Team ID?

In Teams, click "..." next to your team name → "Get link to team". The URL contains `groupId=` parameter which is your Team ID.

### Can I use multiple configurations?

Yes, use the `config_name` parameter to maintain multiple configurations (e.g., "production", "staging", "team-a"). Each configuration is stored separately.

### What authentication type should I choose?

- **Delegated Permissions:** Use when you want actions to appear as performed by a specific user. Requires user authorization.
- **Bot Framework:** Use for automated scenarios and when you need advanced features like approval cards. Bot appears as a separate entity in Teams.

### How often are tokens refreshed?

Microsoft Graph tokens are automatically refreshed every 55 minutes for both authentication types.

### What happens if my authorization code expires?

For Delegated Permissions, once the initial setup is complete and a refresh token is obtained, you don't need the authorization code again. If you need to re-authorize, generate a new authorization code using the OAuth URL.

### What formatting is supported in messages?

Teams messages support HTML formatting. You can use tags like `<b>`, `<i>`, `<h1>`-`<h6>`, `<p>`, `<br>`, `<ul>`, `<ol>`, `<li>`, and more. Markdown is also supported in some contexts.

### Is there a message size limit?

Yes, Teams enforces a 28 KB limit per message. Large messages will be rejected.

---

## Troubleshooting

### "Channel not found" error

**Solution:**
1. Verify channel name matches exactly (case-sensitive)
2. Ensure the Team ID is correct
3. Check that the app/bot is added to the team
4. Verify the service principal has access to the channel
5. Use "General" channel for testing (always exists)
6. Confirm channel type (standard vs private) and appropriate permissions

### "Invalid client secret" or "AADSTS7000215" error

**Solution:**
1. Verify the client secret hasn't expired (check Azure Portal → App registrations → Certificates & secrets)
2. Ensure you copied the secret value (not the Secret ID)
3. Regenerate secret if expired and update in Arambh AI
4. Check that Client ID and Tenant ID are correct (no extra spaces)
5. Verify the app registration is in the correct tenant

### "Insufficient privileges" or "Authorization_RequestDenied"

**Solution:**
1. Ensure API permissions are granted (not just added)
2. Click "Grant admin consent for [Tenant]" in Azure Portal
3. Verify permissions match the required list (Delegated vs Application)
4. Wait 5-10 minutes after granting consent for propagation
5. Check if the user (for delegated auth) has permission to post in the channel

### Bot Framework "Service URL" errors

**Solution:**
1. Verify Service URL matches the bot's messaging endpoint
2. Check bot is properly registered in Azure Bot Service
3. Ensure bot is installed in the target team
4. Verify bot app ID matches client ID in configuration
5. Test bot using Bot Framework Emulator first
6. Confirm Service URL format (should include region)

### Messages not appearing in Teams

**Solution:**
1. Check if message was sent successfully (review API response)
2. Verify you're looking at the correct channel/team
3. Refresh Teams application
4. Check Teams notification settings
5. Review Teams activity log for errors
6. Verify message content doesn't violate Teams policies

### Authorization code issues (Delegated Permissions)

**Solution:**
1. Ensure authorization code is copied completely
2. Use authorization code immediately (expires quickly, typically 10 minutes)
3. Don't include extra characters or spaces
4. Verify redirect URI in request matches Azure AD configuration exactly
5. Generate new authorization code if expired

### Approval cards not working

**Solution:**
1. Confirm you're using Bot Framework authentication (not Delegated)
2. Verify bot is properly installed in the team
3. Check conversation_id is valid and active
4. Ensure bot has permission to send adaptive cards
5. Validate card JSON format
6. Test with simpler card format first

### Team ID not working

**Solution:**
1. Verify Team ID is in GUID format
2. Ensure you extracted groupId (not teamId) from the URL
3. Check if team still exists and is active
4. Verify your app has access to the team
5. Try using Microsoft Graph Explorer to validate Team ID

---

**Related Integrations:**
- [Microsoft Azure Active Directory](microsoft-azure-ad) - User management
- [Microsoft Office 365](microsoft-office365) - Email integration

**Back to:** [Integrations Overview](../)

**Last Updated:** March 2026
