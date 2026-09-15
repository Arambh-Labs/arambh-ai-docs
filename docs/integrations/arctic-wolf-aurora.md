---
layout: default
title: Arctic Wolf Aurora Endpoint Security
permalink: /docs/integrations-guide/arctic-wolf-aurora/
parent: Integrations Guide
nav_order: 8
---

# Arctic Wolf Aurora Endpoint Security

Integration with Arctic Wolf Aurora Endpoint Security for device lookup and endpoint threat investigation.

**Category:** EDR (Endpoint Detection and Response)

**Description:** Retrieve the threats Aurora has found on a device (by device ID or hostname) and run InstaQuery live searches across endpoints for files, processes, network connections, and registry keys.

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [Get Device Threats](#1-get-device-threats)
  - [Run InstaQuery](#2-run-instaquery)
- [FAQ](#faq)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **API Region** | Select | Yes | Regional Aurora API host for your tenant (default: North America). See the region table below. |
| **Tenant ID** | String | Yes | Tenant ID from the Aurora Management Console (**Settings** → **Integrations**) |
| **Application ID** | String | Yes | Application ID of the custom application created under **Settings** → **Integrations** |
| **Application Secret** | Password | Yes | Application Secret of the custom application, used to sign the JWT token request |

### API Regions

| Region | API Host |
|--------|----------|
| North America | `https://protectapi.cylance.com` |
| US Gov | `https://protectapi.us.cylance.com` |
| Europe | `https://protectapi-euc1.cylance.com` |
| APAC North | `https://protectapi-apne1.cylance.com` |
| APAC Southeast | `https://protectapi-au.cylance.com` |
| South America | `https://protectapi-sae1.cylance.com` |

> **Note:** Select the region your Aurora tenant is hosted in. Credentials for one region are rejected by the other regional hosts.

---

## Required Permissions

The custom application must be granted the following privileges in the Aurora Management Console:

- **Devices - Read** - Look up devices by hostname
- **Devices - Threat List** - List the threats found on a device
- **InstaQuery** (create and read) - Required only for the "Run InstaQuery" action

**Authentication Type:** Custom application (Tenant ID, Application ID, and Application Secret). The integration signs a short-lived JWT with the Application Secret and exchanges it for an API access token.

---

## Setup Instructions

### 1. Create a Custom Application

1. Log into the Aurora Management Console as an administrator
2. Navigate to **Settings** → **Integrations**
3. Copy the **Tenant ID** shown on the page
4. Click **Add Application** and enter a name for the application (e.g. `Arambh AI`)
5. Grant the privileges listed in [Required Permissions](#required-permissions)
6. Save the application, then copy the **Application ID** and **Application Secret** — treat the secret as a password and store it securely

### 2. Configure in Arambh AI

- Navigate to Integrations → Arctic Wolf Aurora Endpoint Security
- In the 'Configurations' tab, click on 'Add Configuration'
- Select the **API Region** for your tenant
- Enter the **Tenant ID** and **Application ID**
- Paste the **Application Secret**
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Get Device Threats

Get the threats (malicious or suspicious files) Aurora has found on a device, with each file's classification, score, SHA256, path, and status.

**Action:** `arctic_wolf_aurora--get_device_threats`

#### Parameters

- `device_id` (optional): Aurora unique device ID (a UUID, e.g. `e378dacb-9324-453a-b8c6-5a8406952195`). Takes precedence over `hostname` when both are given.
- `hostname` (optional): Device hostname. For domain-joined machines use the fully qualified name (e.g. `host01.corp.example.com`), otherwise the bare hostname (e.g. `User-Laptop-A123`).
- `limit` (optional): Maximum number of threats to return per device (default: `100`, max: `1000`)
- `config_name` (optional): Configuration profile name

> **Note:** Either `device_id` or `hostname` is required.

#### Example

```json
{
  "hostname": "host01.corp.example.com",
  "limit": 50
}
```

#### Returns

- One entry per matching device, each containing:
  - `device`: A summary of the device — ID, name, hostname, state, safety status, OS version, IP and MAC addresses, last logged-in user, policy, agent version, and offline date. (Populated when the device is looked up by hostname.)
  - `threats`: Array of threat objects including classification, score, SHA256, file path, and file status. Each threat also carries a `file_status_label` — one of `Unsafe`, `Quarantined`, `Whitelisted`, `Suspicious`, `File Removed`, or `Corrupt`.
  - `total_number_of_items`: Total number of threats Aurora reports for the device, which may be higher than the number returned when `limit` is reached

#### Device Lookup

A single hostname can match several Aurora devices (for example, a reimaged machine that registered again). In that case, the threats of each matching device are returned. If no device matches the hostname, the action succeeds with an empty result.

---

### 2. Run InstaQuery

Run an InstaQuery — a live search across endpoints for artifacts Aurora Focus stores locally — and return its results.

**Action:** `arctic_wolf_aurora--run_instaquery`

#### Parameters

- `artifact` (required): Type of artifact to search for — one of `File`, `Process`, `NetworkConnection`, `RegistryKey`
- `match_value_type` (required): The attribute of the artifact to match against. It must be valid for the chosen artifact:

  | Artifact | Valid `match_value_type` values |
  |----------|---------------------------------|
  | `File` | `Path`, `Md5`, `Sha2`, `Owner`, `CreationDateTime` |
  | `Process` | `Name`, `Commandline`, `PrimaryImagePath`, `PrimaryImageMd5`, `StartDateTime` |
  | `NetworkConnection` | `DestAddr`, `DestPort` |
  | `RegistryKey` | `ProcessName`, `ProcessPrimaryImagePath`, `ValueName`, `FilePath`, `FileMd5`, `IsPersistencePoint` |

- `match_values` (required): List of values to match (e.g. `["powershell.exe"]` or `["10.0.0.5", "10.0.0.6"]`)
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "artifact": "NetworkConnection",
  "match_value_type": "DestAddr",
  "match_values": ["10.0.0.5", "10.0.0.6"]
}
```

```json
{
  "artifact": "File",
  "match_value_type": "Sha2",
  "match_values": ["e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"]
}
```

#### Returns

- `query_id`: ID of the InstaQuery created in Aurora
- `completed`: Whether the query finished before the action stopped waiting
- `query_status`, `progress`, `results_available`: Query state as reported by Aurora
- `total_results`: Number of results collected
- `results`: Array of matching artifacts (up to 500), each with the hostname and device ID that reported it, when it was first and last observed, and its properties

---

## FAQ

### Can I use multiple configurations?

Yes, use the `config_name` parameter to maintain multiple configurations (e.g., `production`, `staging`). Each configuration is stored separately.

### How do I find a device ID?

The device ID is shown on the device's page in the Aurora Management Console. You can also call "Get Device Threats" with a hostname — the `device` summary in the response includes the device ID.

---

**Related Integrations:**
- [SentinelOne](sentinel-one) - Alternative EDR platform

**Back to:** [Integrations Overview](../)

**Last Updated:** September 2026
