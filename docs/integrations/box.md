---
layout: default
title: Box
permalink: /docs/integrations-guide/box/
parent: Integrations Guide
nav_order: 6
---

# Box

Integrate with Box enterprise content management platform for file storage, folder management, user administration, and group management.

**Category:** Asset Management

**Description:** Box is an enterprise content management platform that solves simple and complex challenges, from sharing and accessing files.

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [Get Current User](#1-get-current-user)
  - [Get User](#2-get-user)
  - [Create User](#3-create-user)
  - [Update User](#4-update-user)
  - [Delete User](#5-delete-user)
  - [Get File Information](#6-get-file-information)
  - [Upload File](#7-upload-file)
  - [Download File](#8-download-file)
  - [Get Folder Information](#9-get-folder-information)
  - [Create Folder](#10-create-folder)
  - [Move Folder](#11-move-folder)
  - [Create Group](#12-create-group)
  - [Get Group](#13-get-group)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Client ID** | String | Yes | OAuth 2.0 Client ID from the Box Developer Console |
| **Client Secret** | Password | Yes | OAuth 2.0 Client Secret from the Box Developer Console |
| **Enterprise ID** | String | Yes | Box Enterprise ID, found in Admin Console > Account & Billing |
| **Server URL** | String | No | Box API server URL (default: `https://api.box.com`) |

---

## Required Permissions

To use this integration, the Box application requires the following OAuth 2.0 scopes. These are configured in the Box Developer Console under your app's **Configuration** tab:

- **Read and write all files and folders stored in Box** (`root_readwrite`) — required for file and folder operations
- **Manage users** (`manage_managed_users`) — required for user management actions
- **Manage groups** (`manage_groups`) — required for group management actions
- **Manage enterprise properties** (`manage_enterprise_properties`) — required to access enterprise-level data

**Authentication Method:** Server Authentication — Client Credentials Grant (CCG)

The Box app must be configured with **Server Authentication (Client Credentials Grant)** in the Box Developer Console. The app must also be authorized by a Box Admin in the Admin Console under **Apps > Custom Apps**.

---

## Setup Instructions

### 1. Create a Box Application

- Log in to the [Box Developer Console](https://developer.box.com/)
- Click **Create New App**
- Select **Custom App** → **Server Authentication (Client Credentials Grant)**
- Provide an app name (e.g., "Arambh AI Integration") and click **Create App**

### 2. Configure OAuth 2.0 Scopes and Access

- In the app's **Configuration** tab, scroll to **Application Scopes**
- Enable the scopes listed in the Required Permissions section above
- Under **Application Access**, set it to **App + Enterprise Access** — this is required to access and manage Managed Users (existing enterprise users)
- Click **Save Changes**

### 3. Authorize the App in the Admin Console

- Navigate to the **Authorization** tab in the Developer Console and click **Review and Submit** — this sends an authorization request to your Box Admin
- The Box Admin must then approve the app in **Admin Console > Apps > Custom Apps**

### 4. Copy Credentials

- In the app's **Configuration** tab, copy the **Client ID** and **Client Secret**
- The **Enterprise ID** is found in the Admin Console under **Account & Billing > Enterprise Settings**

### 5. Configure in Arambh AI

- Navigate to Integrations → Box
- In the 'Configurations' tab, click on 'Add Configuration'
- Enter the Client ID, Client Secret, and Enterprise ID
- Leave Server URL as default unless using a custom Box instance
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Get Current User

Returns information about the currently authenticated service account user.

**Action:** `box--get_current_user`

#### Parameters

- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "12345678",
  "type": "user",
  "name": "Arambh AI Service Account",
  "login": "AutomationUser@company.com",
  "created_at": "2024-01-01T00:00:00-08:00",
  "status": "active",
  "space_amount": 10737418240,
  "space_used": 0
}
```

---

### 2. Get User

Returns information about a Box user by their user ID.

**Action:** `box--get_user`

#### Parameters

- `user_id` (required): The ID of the Box user
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "12345678",
  "type": "user",
  "name": "John Doe",
  "login": "john.doe@company.com",
  "status": "active",
  "job_title": "Software Engineer",
  "phone": "+1234567890",
  "space_amount": 10737418240,
  "space_used": 524288000
}
```

---

### 3. Create User

Creates a new managed Box user.

**Action:** `box--create_user`

#### Parameters

- `name` (required): Full name of the user
- `login` (required): Email address used as login
- `role` (optional): User role — `coadmin` or `user` (default: `user`)
- `language` (optional): Language preference (e.g. `en`)
- `phone` (optional): Phone number
- `address` (optional): Mailing address
- `job_title` (optional): Job title
- `timezone` (optional): Timezone (e.g. `America/Los_Angeles`)
- `space_amount` (optional): Storage quota in bytes. Use `-1` for unlimited
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "99887766",
  "type": "user",
  "name": "John Doe",
  "login": "john.doe@company.com",
  "status": "active",
  "role": "user",
  "created_at": "2026-03-10T00:00:00-08:00"
}
```

---

### 4. Update User

Updates an existing Box user's information.

**Action:** `box--update_user`

#### Parameters

- `user_id` (required): The ID of the user to update
- `name` (optional): Full name
- `role` (optional): Role — `coadmin` or `user`
- `language` (optional): Language preference
- `phone` (optional): Phone number
- `address` (optional): Mailing address
- `job_title` (optional): Job title
- `timezone` (optional): Timezone
- `space_amount` (optional): Storage quota in bytes. Use `-1` for unlimited
- `status` (optional): Account status — `active`, `inactive`, `cannot_delete_edit`, or `cannot_delete_edit_upload`
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "12345678",
  "type": "user",
  "name": "John Doe",
  "login": "john.doe@company.com",
  "status": "inactive"
}
```

---

### 5. Delete User

Deletes a Box user by their user ID.

**Action:** `box--delete_user`

#### Parameters

- `user_id` (required): The ID of the user to delete
- `notify` (optional): Notify the user by email upon deletion (default: `false`)
- `force` (optional): Delete the user even if they still own files (default: `false`)
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "status": "Success",
  "message": "User deleted successfully"
}
```

---

### 6. Get File Information

Returns the metadata and details of a file stored in Box.

**Action:** `box--get_file_information`

#### Parameters

- `file_id` (required): The ID of the file
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "987654321",
  "type": "file",
  "name": "report.pdf",
  "size": 204800,
  "extension": "pdf",
  "created_at": "2026-03-10T00:00:00-08:00",
  "modified_at": "2026-03-10T00:00:00-08:00",
  "parent": {
    "id": "0",
    "name": "All Files"
  }
}
```

---

### 7. Upload File

Uploads a local file to a Box folder.

**Action:** `box--upload_file`

#### Parameters

- `file_path` (required): Local path to the file to upload (e.g. `/tmp/report.pdf`)
- `name` (required): Name to give the file in Box
- `parent_id` (required): The ID of the parent folder to upload into. Use `0` for the root folder
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "entries": [
    {
      "id": "987654321",
      "type": "file",
      "name": "report.pdf",
      "size": 204800,
      "parent": {
        "id": "0",
        "name": "All Files"
      }
    }
  ]
}
```

---

### 8. Download File

Downloads a file from Box and saves it to `/tmp`. Returns the saved file path and metadata.

**Action:** `box--download_file`

#### Parameters

- `file_id` (required): The ID of the file to download
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "status": "success",
  "data": {
    "file_path": "/tmp/report.pdf",
    "metadata": {
      "name": "report.pdf",
      "type": "pdf",
      "size": 204800
    }
  }
}
```

---

### 9. Get Folder Information

Returns the metadata and contents of a Box folder.

**Action:** `box--get_folder_information`

#### Parameters

- `folder_id` (required): The ID of the folder. Use `0` for the root folder
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "11223344",
  "type": "folder",
  "name": "Incident Evidence",
  "created_at": "2026-03-10T00:00:00-08:00",
  "item_collection": {
    "total_count": 3,
    "entries": []
  }
}
```

---

### 10. Create Folder

Creates a new folder inside an existing Box folder.

**Action:** `box--create_folder`

#### Parameters

- `name` (required): Name of the new folder
- `parent_id` (required): The ID of the parent folder. Use `0` for the root folder
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "11223344",
  "type": "folder",
  "name": "Incident Evidence",
  "created_at": "2026-03-10T00:00:00-08:00",
  "parent": {
    "id": "0",
    "name": "All Files"
  }
}
```

---

### 11. Move Folder

Moves a Box folder to a different parent folder.

**Action:** `box--move_folder`

#### Parameters

- `folder_id` (required): The ID of the folder to move
- `parent_id` (required): The ID of the destination parent folder
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "11223344",
  "type": "folder",
  "name": "Incident Evidence",
  "parent": {
    "id": "55667788",
    "name": "Archived Cases"
  }
}
```

---

### 12. Create Group

Creates a new group in Box.

**Action:** `box--create_group`

#### Parameters

- `name` (required): Name of the group
- `description` (optional): Description of the group
- `invitability_level` (optional): Who can invite users to the group — `admins_only`, `admins_and_members`, or `all_managed_users`
- `member_viewability_level` (optional): Who can view the group members — `admins_only`, `admins_and_members`, or `all_managed_users`
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "98765432",
  "type": "group",
  "name": "Security Team",
  "description": "Security incident response team",
  "created_at": "2026-03-10T00:00:00-08:00"
}
```

---

### 13. Get Group

Returns information about a Box group. If no group ID is provided, returns all groups in the enterprise.

**Action:** `box--get_group`

#### Parameters

- `group_id` (optional): The ID of the group. If omitted, all groups are returned
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "id": "98765432",
  "type": "group",
  "name": "Security Team",
  "description": "Security incident response team",
  "created_at": "2026-03-10T00:00:00-08:00",
  "modified_at": "2026-03-10T00:00:00-08:00"
}
```

---

**Back to:** [Integrations Overview](../)

**Last Updated:** March 2026
