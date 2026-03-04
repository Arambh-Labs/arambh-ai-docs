---
layout: default
title: Microsoft Azure Active Directory
permalink: /docs/integrations-guide/microsoft-azure-ad/
parent: Integrations Guide
nav_order: 1
---

# Microsoft Azure Active Directory

Integrate with Microsoft Azure Active Directory for user management and authentication.

**Category:** Asset Management

**Description:** Query and manage users in your Azure Active Directory tenant using Microsoft Graph API.

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
| **Tenant ID** | String | Yes | Azure AD Tenant ID (GUID format) |
| **Client ID** | String | Yes | Azure AD Application Client ID (GUID format) |
| **Client Secret** | Password | Yes | Azure AD Application Client Secret |

---

## Required API Permissions

To use this integration, your Azure AD application requires the following Microsoft Graph API permissions:

- `User.Read.All` - Read all users' full profiles
- `Directory.Read.All` - Read directory data

**Permission Type:** Application permissions (client credentials flow)

**OAuth Scope:** `https://graph.microsoft.com/.default`

---

## Setup Instructions

### 1. Register an Application in Azure AD

- Go to Azure Portal → Azure Active Directory → App registrations
- Click "New registration"
- Provide a name (e.g., "Arambh AI Integration")
- Select "Accounts in this organizational directory only"
- Click "Register"

### 2. Configure API Permissions

- Navigate to "API permissions"
- Click "Add a permission" → Microsoft Graph → Application permissions
- Add `User.Read.All` and `Directory.Read.All`
- Click "Grant admin consent" (requires admin privileges)

### 3. Create Client Secret

- Navigate to "Certificates & secrets"
- Click "New client secret"
- Provide a description and select expiration period
- Copy the secret value (only shown once)

### 4. Configure in Arambh AI

- Navigate to Integrations → Azure Active Directory
- In the 'Configurations' tab, click on 'Add Configuration'
- Enter Tenant ID (from Overview page)
- Enter Client ID (Application ID from Overview page)
- Enter Client Secret (copied in previous step)
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Lookup User

Look up a specific user in Azure AD by their identifier.

**Action:** `microsoft_azureactivedirectory--lookup_user`

#### Parameters

- `user_identifier` (required): User's userPrincipalName, email address, or Azure AD object ID
- `select_fields` (optional): Comma-separated list of fields to return
  - Default: `id,displayName,userPrincipalName,givenName,surname,mail,jobTitle,department,companyName,mobilePhone,businessPhones,officeLocation,accountEnabled,userType,lastPasswordChangeDateTime`
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

- User object with selected fields
- For search queries: Array of matching users with count

#### Example Response

```json
{
  "id": "12345-67890-abcde",
  "displayName": "John Doe",
  "userPrincipalName": "john.doe@company.com",
  "mail": "john.doe@company.com",
  "jobTitle": "Software Engineer",
  "department": "Engineering",
  "accountEnabled": true
}
```

---

### 2. List Users

Retrieve a list of users with optional filtering.

**Action:** `microsoft_azureactivedirectory--list_users`

#### Parameters

- `filter_query` (optional): OData filter query (e.g., `accountEnabled eq true`)
- `select_fields` (optional): Comma-separated list of fields to return
  - Default: `id,displayName,userPrincipalName,mail,jobTitle,department,accountEnabled`
- `top` (optional): Maximum number of users to return (max: 999, default: 10)
- `config_name` (optional): Configuration profile name

#### Common Filter Examples

- `accountEnabled eq true` - Only active users
- `department eq 'Engineering'` - Users in specific department
- `startswith(displayName,'John')` - Users whose name starts with "John"

#### Returns

- Array of user objects
- Total count
- Pagination links if more results available

---

**Related Integrations:**
- [Microsoft Office 365](microsoft-office365) - Email integration
- [Microsoft Teams](microsoft-teams) - Teams messaging

**Back to:** [Integrations Overview](../)

**Last Updated:** March 2026
