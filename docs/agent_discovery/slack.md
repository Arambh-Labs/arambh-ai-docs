---
layout: default
title: Slack Discovery
permalink: /docs/agent-discovery-guide/slack/
parent: Agent Discovery Guide
nav_order: 6
---

# Slack Discovery

Inventory workspace apps (and optionally services) from Slack integration logs to understand OAuth blast radius in Slack.

**Toolkit name:** `slack`

---

## Table of Contents

- [Configuration](#configuration)
- [Required permissions](#required-permissions)
- [Setup instructions](#setup-instructions)

---

## Configuration

| Field           | Type     | Required | Description                                                                                    |
| --------------- | -------- | -------- | ---------------------------------------------------------------------------------------------- |
| **Bot token**   | Password | Yes      | Bot User OAuth Token — used for notifications; fallback for discovery if no admin token        |
| **Channel ID**  | String   | Yes      | Default channel for notifications (not required for discovery itself)                          |
| **Admin token** | Password | No       | User token (`xoxp-...`) with admin access for `team.integrationLogs`. Preferred for discovery. |

---

## Required permissions

1. Create and install a Slack app for **notifications** (bot token + channel).
2. For blast radius discovery, provide an **admin-scoped user token** that can call `team.integrationLogs`, **or** ensure the bot token includes the Slack **admin** capability required for that method.

Without admin integration-log access, discovery cannot enumerate workspace apps from history.

---

## Setup instructions

### 1. Create a Slack app (notifications)

1. [api.slack.com/apps](https://api.slack.com/apps) → Create New App
2. Add bot scopes needed for posting messages; install to the workspace
3. Copy the **Bot User OAuth Token** and a target **Channel ID**

### 2. Add an admin token for discovery

1. Use a workspace admin user token (`xoxp-...`) authorized for integration logs  
   **or** ensure the bot has the required admin scope for `team.integrationLogs`
2. Store it in the **Admin token** field

### 3. Configure in Arambh AI

1. Integrations → Slack → **Add Configuration**
2. Enter bot token, channel ID, and optional admin token
3. Save and run **Healthcheck**

---

**Last Updated:** July 2026
