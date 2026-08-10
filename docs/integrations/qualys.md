---
layout: default
title: Qualys
permalink: /docs/integrations-guide/qualys/
parent: Integrations Guide
nav_order: 13
---

# Qualys

Manage IP assets, discover hosts and asset groups, look up vulnerabilities, and launch/track VM and PC scans on the Qualys Cloud Platform.

**Category:** Asset Management

**Description:** Add, list, and update IP assets in the Vulnerability Management (VM) and Policy Compliance (PC) modules; discover scanned hosts, asset groups, and virtual hosts; search the KnowledgeBase for vulnerabilities; and launch, list, and fetch results for VM and PC scans — all against the classic Qualys VM/PC API (`/api/2.0/fo/...`).

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
- [Authentication Notes](#authentication-notes)
- [Known Limitations](#known-limitations)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Server URL** | String | Yes | API Server URL for your Qualys platform, e.g. `https://qualysapi.qg1.apps.qualys.in`. This depends on which Qualys platform (pod) your subscription is on — see [Setup Instructions](#setup-instructions). |
| **Username** | String | Yes | Your Qualys **Login ID** — not your email address and not your display name. See [Setup Instructions](#setup-instructions). |
| **Password** | Password | Yes | Password for the Qualys user account above. |
| **Verify SSL** | String | No | Verify the SSL certificate of the Qualys server. `"true"` or `"false"`. Defaults to `true` if unset. |

---

## Required Permissions

Qualys permissions are role-based and enforced per action:

| Permission | Required For |
|-----------|-------------|
| Any valid, API-enabled login | `get_ip_list`, `get_scanned_host`, `get_asset_groups`, `get_host_list`, `search_vulnerability`, `get_vm_scan_list`, `get_pc_scan_list` |
| **"Add assets"** | `add_ip` (VM module) |
| **"Add assets" + "Manage compliance"** | `add_ip` with `add_to_pc: true` |
| Manager, or Unit Manager scoped to the target hosts | `update_ip` |
| Manager or Unit Manager | `launch_vm_scan`, `launch_pc_scan` |

The Qualys account also needs **API Access** enabled (a per-user flag, separate from GUI/console access — see **Users → [user] → Edit → User Role → API** in the Qualys UI), and the subscription tier itself must include API access at all (see [Known Limitations](#known-limitations)).

---

## Setup Instructions

### 1. Find Your Server URL

Qualys hosts API traffic on a different subdomain per platform/region — using the wrong one will fail even with correct credentials.

**Easiest — from the UI:** Log into the Qualys web console → **Help → About**. Under **Security Operations Center (SOC)** you'll see the exact **API Server URL** for your account. Use that value as-is.

**Important:** the SOC page also lists a `qualysguard.*` hostname (the web console) alongside the `qualysapi.*` hostname (the API). These are easy to mix up since they share the same platform suffix — **only the `qualysapi.*` one works for API calls.**

**Reference table** (platform identifier is embedded in your Qualys username):

| Platform | API Server URL |
|---|---|
| US1 | `https://qualysapi.qualys.com` |
| US2 | `https://qualysapi.qg2.apps.qualys.com` |
| US3 | `https://qualysapi.qg3.apps.qualys.com` |
| US4 | `https://qualysapi.qg4.apps.qualys.com` |
| EU1 | `https://qualysapi.qualys.eu` |
| EU2 | `https://qualysapi.qg2.apps.qualys.eu` |
| EU3 | `https://qualysapi.qg3.apps.qualys.it` |
| IN1 | `https://qualysapi.qg1.apps.qualys.in` |
| CA1 | `https://qualysapi.qg1.apps.qualys.ca` |
| AE1 | `https://qualysapi.qg1.apps.qualys.ae` |
| UK1 | `https://qualysapi.qg1.apps.qualys.co.uk` |
| AU1 | `https://qualysapi.qg1.apps.qualys.com.au` |
| KSA1 | `https://qualysapi.qg1.apps.qualysksa.com` |

### 2. Find Your Login ID (not your email)

Qualys authenticates on a **Login ID** — a short username Qualys assigns you — which is often *different* from the email address or display name shown on your account profile.

Go to **Users → User Accounts** (list view, not the edit-user panel) — there's a **"Login"** column showing the real login ID for each user (e.g. `nne8gm`). Use that exact value, not the email address.

### 3. Confirm API Access Is Actually Available

Before configuring, confirm your subscription tier actually includes API access — **Qualys Community Edition (free trial) does not include any API access**, regardless of what the per-user "API Access" toggle shows (see [Known Limitations](#known-limitations)). If you're on Community Edition, none of these actions will work until you upgrade to Express Lite or a paid subscription.

### 4. Configure in Arambh AI

- Navigate to **Integrations → Qualys**
- In the **Configurations** tab, click **Add Configuration**
- Enter **Server URL**, **Username** (Login ID), and **Password**
- Click **Save**
- In the **Actions** tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Add Asset

Adds IP addresses to the subscription, optionally enabling them for the VM and/or PC module.

**Action:** `qualys--add_ip`

#### Parameters

- `ips` (required): IP addresses to add — comma-separated, ranges (`10.10.10.1-10.10.10.10`), or CIDR (`10.10.10.0/24`)
- `add_to_vm` (optional): Add to the Vulnerability Management module. Default `true`
- `add_to_pc` (optional): Add to the Policy Compliance module. Default `false`. At least one of `add_to_vm`/`add_to_pc` must be true
- `tracking_method` (optional): `IP`, `DNS`, or `NETBIOS`. Default `IP`
- `owner` (optional): Asset owner (Manager or Unit Manager)
- `ag_title` (optional): Asset group title — required if the API user is a Unit Manager
- `ud1` / `ud2` / `ud3` (optional): User-defined attribute fields
- `comment` (optional): Free-text comment
- `config_name` (optional): Configuration profile name

#### Example

```json
{
  "ips": "10.10.10.1-10.10.10.10",
  "add_to_vm": true,
  "tracking_method": "IP"
}
```

---

### 2. Get Asset List

Retrieves IP addresses in the account, optionally filtered.

**Action:** `qualys--get_ip_list`

#### Parameters

- `ips` (optional): Restrict to specific IPs/ranges/CIDR blocks. Omit to list all
- `tracking_method` (optional): `IP`, `DNS`, or `NETBIOS`
- `network_id` (optional): Restrict to a custom network ID (requires Network Support feature)
- `compliance_enabled` (optional): Only IPs enabled for the PC module
- `config_name` (optional): Configuration profile name

---

### 3. Update Asset

Updates attributes of existing IP assets.

**Action:** `qualys--update_ip`

#### Parameters

- `ips` (required): IP address(es) to update
- `tracking_method` (optional): `IP`, `DNS`, or `NETBIOS`. Cannot change to/from EC2 or Agent tracking
- `host_dns` / `host_netbios` (optional): New hostname — requires a single IP in `ips`
- `network_id` (optional): Restrict to a custom network ID
- `owner` (optional): New asset owner
- `ud1` / `ud2` / `ud3` (optional): User-defined attribute fields
- `comment` (optional): Free-text comment
- `config_name` (optional): Configuration profile name

---

### 4. Get Scanned Host List

Retrieves detailed information on scanned hosts.

**Action:** `qualys--get_scanned_host`

#### Parameters

- `ips` / `ids` / `ag_titles` (optional): Filter by IPs, host IDs, or asset group titles
- `details` (optional): `Basic`, `Basic/AGs`, `All`, `All/AGs`, or `None`. Default `Basic`
- `vm_scan_since` / `no_vm_scan_since` (optional): Date filters (`YYYY-MM-DD[THH:MM:SSZ]`)
- `compliance_enabled` (optional): Only hosts enabled for PC
- `os_pattern` (optional): PCRE regex to filter by OS
- `truncation_limit` (optional): Max records per response (default 1000 on the Qualys side)
- `config_name` (optional): Configuration profile name

---

### 5. Get Asset Group List

Retrieves asset groups.

**Action:** `qualys--get_asset_groups`

#### Parameters

- `ids` / `id_min` / `id_max` (optional): Filter by asset group ID(s)
- `network_ids` (optional): Restrict to network IDs
- `unit_id` / `user_id` (optional): Filter by owning business unit or user
- `show_attributes` (optional): `None`, `All`, or comma-separated attribute names (e.g. `TITLE,OWNER,IP_SET`)
- `truncation_limit` (optional): Max records per response
- `config_name` (optional): Configuration profile name

---

### 6. Get Virtual Host List

Retrieves virtual hosts (IP + port + FQDN mappings).

**Action:** `qualys--get_host_list`

#### Parameters

- `ip` (optional): Filter by IP address
- `port` (optional): Filter by port
- `config_name` (optional): Configuration profile name

---

### 7. Get Vulnerability List

Searches the Qualys KnowledgeBase for vulnerabilities.

**Action:** `qualys--search_vulnerability`

#### Parameters

- `ids` (optional): QIDs or QID ranges (comma-separated)
- `details` (optional): `Basic`, `All`, or `None`. Default `Basic`
- `is_patchable` (optional): Filter to patchable or non-patchable vulnerabilities
- `published_after` / `published_before` (optional): Date filters
- `last_modified_after` / `last_modified_before` (optional): Date filters
- `config_name` (optional): Configuration profile name

#### Notes

- **This endpoint filters by QID, not CVE ID.** Qualys' KnowledgeBase list API has no native CVE filter parameter, despite what some third-party connector changelogs suggest. To find vulnerabilities for a known CVE, retrieve results and inspect the returned `CVE_ID_LIST` field, or use a threat-intel lookup to resolve CVE → QID first.

---

### 8 & 9. Launch VM Scan / Launch PC Scan

Launches a Vulnerability Management or Policy Compliance scan. Both actions share one internal implementation — Qualys' launch-scan request shape is identical between VM and PC, differing only in the base API path (`/fo/scan/` vs `/fo/scan/compliance/`).

**Actions:** `qualys--launch_vm_scan`, `qualys--launch_pc_scan`

#### Parameters

- `scan_title` (required): Title for the scan
- `ip` / `asset_groups` (one required when `target_from` is `assets`): Target IPs or asset group titles
- `target_from` (optional): `assets` or `tags`. Default `assets`
- `iscanner_name` (optional): Scanner appliance name(s). Omit to use the default/external scanner
- `option_title` (optional): Option profile title
- `priority` (optional): 0 (default, no priority) to 9 (highest)
- `config_name` (optional): Configuration profile name

---

### 10 & 11. Get VM Scan List / Get PC Scan List

Lists scans, optionally filtered. Same shared-implementation pattern as launch scan.

**Actions:** `qualys--get_vm_scan_list`, `qualys--get_pc_scan_list`

#### Parameters

- `scan_ref` (optional): A specific scan reference (e.g. `scan/1234567890.12345`)
- `state` (optional): `Running`, `Paused`, `Canceled`, `Finished`, `Error`, `Queued`, or `Loading`
- `scan_run_type` (optional): `On-Demand`, `Scheduled`, or `API`
- `launched_after_datetime` / `launched_before_datetime` (optional): Date filters. When both omitted, Qualys defaults to the last 30 days
- `config_name` (optional): Configuration profile name

---

### 12 & 13. Fetch VM Scan / Fetch PC Scan

Downloads scan results for a `Finished`, `Canceled`, `Paused`, or `Error` scan.

**Actions:** `qualys--fetch_vm_scan`, `qualys--fetch_pc_scan`

#### Parameters

- `scan_ref` (required): The scan reference to fetch
- `output_format`, `ips`, `mode` (optional, **VM scans only**): Format (`csv`/`json`/`csv_extended`/`json_extended`), IP filter, and verbosity (`brief`/`extended`). Not applicable to PC scan fetches
- `config_name` (optional): Configuration profile name

---

## Authentication Notes

This integration authenticates using Qualys' **session login/logout flow** (`POST /api/2.0/fo/session/?action=login`) rather than HTTP Basic Authentication, even though Basic Auth is the more commonly documented method in Qualys' quick-reference examples. This is deliberate: on the account this integration was originally built and tested against, Basic Auth consistently returned `Bad Login/Password` for verified-correct credentials, while session-based login succeeded with the same credentials.

The session cookie is cached in the toolkit config (encrypted, alongside the password) and reused across calls rather than re-authenticating on every request, since Qualys explicitly warns that excessive session logins can lock out further login attempts. On a `401` response, the integration transparently re-logs in once and retries before failing.

If your subscription has Basic Auth enabled and working, this won't cause any issues — session-based auth is a fully supported, first-class Qualys authentication method for all `/api/2.0/` endpoints.

---

## Known Limitations

**Qualys Community Edition (free trial) does not support API access at all**, independent of anything in this integration or its configuration. This is stated explicitly in Qualys' own documentation:

> Source: [Qualys Community Edition User Guide](https://cdn2.qualys.com/docs/qualys-community-edition-user-guide.pdf), section **"Community Edition vs. Express Lite"**
>
> | | Community Edition | Express Lite |
> |---|---|---|
> | **API compatible** | No | Yes |

If your account is on Community Edition:
- The Qualys web UI will work normally (GUI access is included).
- The per-user **"API Access: Yes"** toggle in the Qualys UI will still show as enabled — this is misleading; it reflects a permission that would matter on an API-capable tier, but is not functionally granted on Community Edition.
- Every action in this integration will fail authentication (`Bad Login/Password`, `CODE 2000`), regardless of correct credentials, correct server URL, or auth method (Basic Auth and session auth both fail identically).

**This integration was built and cross-verified against Qualys' official API documentation (parameter-by-parameter, across every action) but could not be end-to-end tested against live data**, because the only account available during development was Community Edition. The request-building code is confirmed correct; whether it works end-to-end against a real subscription has not been empirically verified. If you're setting this up against a paid subscription (Express Lite or higher) and hit unexpected behavior, it's more likely to be a genuine integration bug than anything found during development — please report it.

To get API access: upgrade from Community Edition to Express Lite or a full Qualys subscription (contact your Qualys Technical Account Manager or Qualys Support).

---

**Back to:** [Integrations Overview](../)

**Last Updated:** August 2026
