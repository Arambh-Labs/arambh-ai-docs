---
layout: default
title: Google Chronicle SecOps
permalink: /docs/integrations-guide/google-chronicle-secops/
parent: Integrations Guide
nav_order: 6
---

# Google Chronicle SecOps

Integrate with Google Security Operations (Chronicle SIEM) for UDM search, alert management, and direct API access.

**Category:** SIEM and Datalake

**Description:** Google Security Operations (Chronicle) is a cloud-native SIEM that provides UDM-based threat detection, alert management, and broad API access across the Chronicle platform.

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [UDM Search](#1-udm-search)
  - [Fetch Alerts View](#2-fetch-alerts-view)
  - [Get Alert](#3-get-alert)
  - [Execute API](#4-execute-api)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **GCP Project ID** | String | Yes | Google Cloud project ID associated with your Chronicle instance |
| **Instance ID** | String | Yes | Google SecOps SIEM instance ID |
| **Regional Endpoint** | Select | Yes | Region where your Chronicle instance is hosted (default: `us`) |
| **Service Account JSON Path** | String | Yes | File path to the Google service account JSON key file on the server |

### Available Regional Endpoints

`asia-northeast1`, `asia-south1`, `asia-southeast1`, `australia-southeast1`, `eu`, `europe`, `europe-west12`, `europe-west2`, `europe-west3`, `europe-west6`, `europe-west9`, `me-central1`, `me-central2`, `me-west1`, `northamerica-northeast2`, `southamerica-east1`, `us`

---

## Required Permissions

### GCP IAM Role

Assign one of the following predefined roles to the service account:

| Role | Use case |
|------|----------|
| `roles/chronicle.viewer` | Minimum — read-only access (UDM search, fetch alerts, get alert) |
| `roles/chronicle.editor` | Required if using `execute_api` for write operations (POST/PUT/DELETE) |

### OAuth Scopes

The service account authenticates using the following OAuth scopes:

- `https://www.googleapis.com/auth/chronicle-backstory` — Legacy backstory API domain (used by `execute_api` with `api_domain=backstory`)
- `https://www.googleapis.com/auth/malachite-ingestion` — Chronicle ingestion API
- `https://www.googleapis.com/auth/cloud-platform` — Broad GCP platform access

### IAM Permissions Used

| Action | IAM Permission |
|--------|---------------|
| UDM Search | `chronicle.events.udmSearch` |
| Fetch Alerts View | `chronicle.legacies.legacyFetchAlertsView` |
| Get Alert | `chronicle.legacies.legacyGetAlert` |
| Execute API | Depends on the endpoint called |

---

## Setup Instructions

### 1. Create a GCP Service Account

- Go to [Google Cloud Console](https://console.cloud.google.com) → IAM & Admin → Service Accounts
- Click **Create Service Account**
- Provide a name (e.g., `arambh-chronicle-integration`)
- Click **Create and Continue**

### 2. Assign IAM Role

- Under **Grant this service account access to project**, assign:
  - `Chronicle API Viewer` (`roles/chronicle.viewer`) for read-only operations
  - `Chronicle API Editor` (`roles/chronicle.editor`) if write operations via `execute_api` are needed
- Click **Continue** → **Done**

### 3. Generate Service Account Key

- Click on the created service account
- Navigate to the **Keys** tab
- Click **Add Key** → **Create new key** → Select **JSON**
- Download the JSON key file and place it on the server at a secure path
- Note the full file path (e.g., `/etc/arambh/credentials/chronicle-sa.json`)

### 4. Find Your Instance ID and Project ID

- In Google Cloud Console, navigate to **Security** → **Chronicle**
- Your **Project ID** is visible in the top project selector
- The **Instance ID** is found in Chronicle settings or provided by Google during onboarding

### 5. Configure in Arambh AI

- Navigate to Integrations → Google Chronicle SecOps
- In the **Configurations** tab, click **Add Configuration**
- Enter GCP Project ID
- Enter Instance ID
- Enter the Regional Endpoint matching your Chronicle instance
- Enter the Service Account JSON Path (full path on the server)
- Click **Save** to save the configuration
- In the **Actions** tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. UDM Search

Execute a UDM (Unified Data Model) search query against your Chronicle instance.

**Action:** `googlesecops--udm_search`

**Endpoint:** `POST https://{region}-chronicle.googleapis.com/v1alpha/projects/{project}/locations/{region}/instances/{instance}:udmSearch`

#### Parameters

- `query` (required): UDM query string (e.g., `security_result.alert_state = "ALERTING"`)
- `start_time` (required): Start of search window in RFC3339 format (e.g., `2024-01-01T00:00:00Z`)
- `end_time` (required): End of search window in RFC3339 format (e.g., `2024-01-02T00:00:00Z`)
- `limit` (optional): Maximum number of results to return (default: `100`)
- `config_name` (optional): Configuration profile name (default: `"googlesecopssoar"`)

#### Example

```json
{
  "query": "security_result.alert_state = \"ALERTING\"",
  "start_time": "2024-01-01T00:00:00Z",
  "end_time": "2024-01-02T00:00:00Z",
  "limit": 50
}
```

#### Returns

```json
{
  "events": [...],
  "moreDataAvailable": false,
  "totalCount": 12
}
```

---

### 2. Fetch Alerts View

Fetch legacy alerts from Chronicle within a specified time range.

**Action:** `googlesecops--fetch_alerts_view`

**Endpoint:** `GET https://{region}-chronicle.googleapis.com/v1alpha/projects/{project}/locations/{region}/instances/{instance}/legacy:legacyFetchAlertsView`

#### Parameters

- `start_time` (required): Start time in ISO format (e.g., `2024-01-01T00:00:00Z`)
- `end_time` (required): End time in ISO format
- `baseline_query` (optional): Additional query filter to narrow results
- `limit` (optional): Maximum number of alerts to return (default: `100`)
- `config_name` (optional): Configuration profile name (default: `"googlesecopssoar"`)

#### Example

```json
{
  "start_time": "2024-01-01T00:00:00Z",
  "end_time": "2024-01-02T00:00:00Z",
  "baseline_query": "severity = HIGH",
  "limit": 25
}
```

#### Returns

```json
{
  "alerts": [...],
  "nextPageToken": "..."
}
```

---

### 3. Get Alert

Retrieve details of a specific alert by its alert ID.

**Action:** `googlesecops--get_alert`

**Endpoint:** `GET https://{region}-chronicle.googleapis.com/v1alpha/projects/{project}/locations/{region}/instances/{instance}/legacy:legacyGetAlert`

#### Parameters

- `alert_id` (required): Alert ID (e.g., `sa_12345678-1234-5678-1234-567812345678`)
- `include_detections` (optional): Include detection details in the response (default: `false`)
- `config_name` (optional): Configuration profile name (default: `"googlesecops"`)

#### Example

```json
{
  "alert_id": "sa_12345678-1234-5678-1234-567812345678",
  "include_detections": true
}
```

#### Returns

```json
{
  "alert": {
    "id": "sa_12345678-1234-5678-1234-567812345678",
    "type": "RULE_DETECTION",
    "detections": [...]
  }
}
```

---

### 4. Execute API

Execute arbitrary Chronicle API calls across the `chronicle` or `backstory` domains with any HTTP method.

**Action:** `googlesecops--execute_api`

**Endpoint:** `https://{region}-{api_domain}.googleapis.com{endpoint}`

#### Parameters

- `api_domain` (required): API domain to call — `"chronicle"` or `"backstory"`
- `endpoint` (required): API endpoint path (e.g., `/v1/events:udmSearch`)
- `method` (optional): HTTP method — `GET`, `POST`, `PUT`, or `DELETE` (default: `GET`)
- `params` (optional): Query parameters as a JSON object
- `data` (optional): Request body as a JSON object
- `config_name` (optional): Configuration profile name (default: `"googlesecops"`)

#### Example

```json
{
  "api_domain": "chronicle",
  "endpoint": "/v1alpha/projects/my-project/locations/us/instances/my-instance/rules",
  "method": "GET"
}
```

#### Returns

Response body from the Chronicle API as a JSON object.

**Note:** Write operations (POST/PUT/DELETE) require the service account to have `roles/chronicle.editor` or higher.

---

**Back to:** [Integrations Overview](../)

**Last Updated:** April 2026
