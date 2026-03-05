---
layout: default
title: Integrations Guide
permalink: /docs/integrations-guide/
parent: Documentation
nav_order: 2
---

# Integrations Guide

Learn how to integrate Arambh AI with various third-party services and platforms.

---

## Integration Categories

### 🔒 Security & SIEM

Security Information and Event Management platforms for threat detection and analysis.

| Integration | Description | Actions |
|-------------|-------------|---------|
| [**IBM QRadar**](ibm-qradar) | Query offenses, run AQL queries, and analyze security events | Get Offenses, Get Events, Run Query, Get IPs |

### 🔥 Network Security

Firewall and network security platforms for traffic control and threat prevention.

| Integration | Description | Actions |
|-------------|-------------|---------|
| [**Fortinet FortiGate**](fortigate-firewall) | Manage host quarantine, block/unblock IPs, applications, and URLs on FortiGate firewalls | Quarantine Host, Unquarantine Host, Block IP, Unblock IP, Block Application, Unblock Application, Block URL, Unblock URL |

### 🛡️ Endpoint Detection & Response

Endpoint security platforms for threat detection, investigation, and response.

| Integration | Description | Actions |
|-------------|-------------|---------|
| [**SentinelOne**](sentinel-one) | Investigate threats, query Deep Visibility events, and retrieve cloud detection alerts | Get Hash Details, Get Threat Details, Fetch Threats, Get Threat Events, Get Events, Get Events By Type, Get Alerts |

### 💬 Communication & Collaboration

Tools for notifications, alerts, and team collaboration.

| Integration | Description | Actions |
|-------------|-------------|---------|
| [**Microsoft Teams**](microsoft-teams) | Send messages, replies, and approval cards to Teams channels | Send Message, Reply, Chat, Approval Card |
| [**Microsoft Office 365**](microsoft-office365) | Send and manage emails with attachments | Send Email, Search Emails, Reply to Email |

### 👥 Identity & Access Management

User and identity management platforms.

| Integration | Description | Actions |
|-------------|-------------|---------|
| [**Microsoft Azure Active Directory**](microsoft-azure-ad) | Query and manage users in Azure AD | Lookup User, List Users |

---

## Getting Started

### General Authentication Flow

Most integrations follow a standard authentication pattern:

1. **Register Application** or obtain API credentials
2. **Configure Permissions** as specified in integration documentation
3. **Create Secrets/Tokens** for secure authentication
4. **Configure in Arambh AI** with your credentials
5. **Test Connection** to verify setup

---

## FAQ

### How many integrations can I enable simultaneously?
You can enable as many integrations as needed. There's no limit on the number of active integrations.

### Are integrations available on all pricing plans?
Yes, integrations are available on all plans.

### How do I disconnect an integration?
Navigate to the Integrations page, find the integration you want to remove, and click "Disable".

### Can I use multiple configurations for the same integration?
Yes, you can add multiple configurations, and give them a unique name.
---

## Troubleshooting
TDB

## Need More Help?

### Support Channels
- **Email Support:** support@arambhlabs.com
- **Integrations Team:** integrations@arambhlabs.com

---

**Last Updated:** March 2026
