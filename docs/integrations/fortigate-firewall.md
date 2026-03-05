---
layout: default
title: Fortinet FortiGate
permalink: /docs/integrations-guide/fortigate-firewall/
parent: Integrations Guide
nav_order: 5
---

# Fortinet FortiGate

Integrate with Fortinet FortiGate enterprise firewall for network security management, host quarantine, and IP address blocking.

**Category:** Network Security

**Description:** Fortinet FortiGate enterprise firewall provides high performance, consolidated advanced security and granular visibility for broad protection across the entire digital attack surface.

---

## Table of Contents
- [Configuration](#configuration)
- [Required Permissions](#required-permissions)
- [Setup Instructions](#setup-instructions)
- [Supported Actions](#supported-actions)
  - [Quarantine Host](#1-quarantine-host)
  - [Unquarantine Host](#2-unquarantine-host)
  - [Block IP](#3-block-ip)
  - [Unblock IP](#4-unblock-ip)
  - [Block Application](#5-block-application)
  - [Unblock Application](#6-unblock-application)
  - [Block URL](#7-block-url)
  - [Unblock URL](#8-unblock-url)

---

## Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **Hostname** | String | Yes | IP address or hostname of the FortiGate server (e.g. `https://192.168.1.1` or `192.168.1.1`) |
| **Port** | Integer | Yes | Port number for connecting to the FortiGate server (default: `443`) |
| **API Key** | Password | Yes | REST API key for authentication |
| **VDOM** | String | No | Virtual domain name, if VDOM mode is enabled. Leave empty to use the root VDOM. |

---

## Required Permissions

To use this integration, the API admin profile on FortiGate requires the following minimum permissions:

- **User and Device** - Read and Write
- **Firewall** - Read and Write (Policy, Address, and Service)
- **Security Profile** - Read and Write (Web Filter and Application Control)
- **Security Fabric** - Read and Write
- **WiFi & Switch** - Read and Write
- **Log & Report** - Read and Write

---

## Setup Instructions

### 1. Create a REST API Administrator

- Log in to the FortiGate web interface
- Navigate to **System > Administrators**
- Click **Create New > REST API Admin**
- Provide a username and assign an admin profile with the permissions listed above
- Under **Trusted Hosts**, restrict access to specific IP ranges if needed
- Click **OK** — the API key is displayed **once**, copy and store it securely

### 2. Enable VDOM Mode (if applicable)

- Navigate to **System > Settings**
- Enable **Virtual Domains (VDOMs)** if managing multiple virtual domains
- Note the VDOM name(s) you want to target with this integration

### 3. Configure in Arambh AI

- Navigate to Integrations → Fortinet FortiGate
- In the 'Configurations' tab, click on 'Add Configuration'
- Enter the Hostname (with or without `https://`)
- Enter the Port (default: 443)
- Enter the API Key (copied in previous step)
- Enter the VDOM name if applicable
- Click "Save" to save the configuration
- In the 'Actions' tab, enable the actions you want the platform to use

---

## Supported Actions

### 1. Quarantine Host

Quarantine one or more hosts on FortiGate by their MAC addresses.

**Action:** `fortigate_firewall--quarantine_host`

#### Parameters

- `macs` (required): Comma-separated list of MAC addresses to quarantine (e.g. `00:11:22:33:44:55, AA:BB:CC:DD:EE:FF`)
- `vdom` (optional): Virtual domain name to target
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "already_quarantine": ["00:11:22:33:44:55"],
  "newly_quarantine": ["AA:BB:CC:DD:EE:FF"],
  "not_quarantine": []
}
```

---

### 2. Unquarantine Host

Remove one or more hosts from quarantine on FortiGate by their MAC addresses.

**Action:** `fortigate_firewall--unquarantine_host`

#### Parameters

- `macs` (required): Comma-separated list of MAC addresses to remove from quarantine
- `vdom` (optional): Virtual domain name to target
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "not_exist": [],
  "newly_unquarantine": ["AA:BB:CC:DD:EE:FF"],
  "not_unquarantine": []
}
```

---

### 3. Block IP

Block one or more IP addresses on FortiGate using the quarantine-based ban mechanism.

**Action:** `fortigate_firewall--block_ip`

#### Parameters

- `ip_addresses` (required): Comma-separated list of IP addresses to block (e.g. `192.168.1.1, 10.0.0.5`)
- `expiry` (optional): Time in seconds before the ban expires. Use `0` to never expire (default: `0`)
- `vdom` (optional): Virtual domain name to target
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "already_blocked": ["192.168.1.1"],
  "newly_blocked": ["10.0.0.5"],
  "error_with_block": []
}
```

---

### 4. Unblock IP

Remove one or more IP addresses from the FortiGate ban list.

**Action:** `fortigate_firewall--unblock_ip`

#### Parameters

- `ip_addresses` (required): Comma-separated list of IP addresses to unblock
- `vdom` (optional): Virtual domain name to target
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "ip_not_exist": [],
  "newly_unblocked": ["10.0.0.5"],
  "error_with_unblock": []
}
```

---

### 5. Block Application

Block one or more applications on FortiGate by adding them to an application control profile.

**Action:** `fortigate_firewall--block_application`

#### Parameters

- `app_names` (required): Comma-separated list of application names to block (e.g. `Facebook, BitTorrent`)
- `app_block_policy` (required): Name of the application control profile to update
- `vdom` (optional): Virtual domain name to target
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
[
  {"name": "Facebook", "message": "Application blocked successfully", "status": "Successful"},
  {"name": "BitTorrent", "message": "Application already blocked", "status": "Successful"}
]
```

---

### 6. Unblock Application

Unblock one or more applications on FortiGate by removing them from an application control profile.

**Action:** `fortigate_firewall--unblock_application`

#### Parameters

- `app_names` (required): Comma-separated list of application names to unblock
- `app_block_policy` (required): Name of the application control profile to update
- `vdom` (optional): Virtual domain name to target
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
[
  {"name": "Facebook", "message": "Application unblocked successfully", "status": "Successful"},
  {"name": "BitTorrent", "message": "Application not found in block state", "status": "Successful"}
]
```

---

### 7. Block URL

Block one or more URLs on FortiGate by adding them to a web filter profile.

**Action:** `fortigate_firewall--block_url`

#### Parameters

- `urls` (required): Comma-separated list of URLs to block (e.g. `example.com, social-media.com/path`)
- `url_filter_profile` (required): Name of the web filter profile to update
- `vdom` (optional): Virtual domain name to target
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "already_blocked": ["example.com"],
  "newly_blocked": ["social-media.com/path"],
  "not_block": []
}
```

---

### 8. Unblock URL

Unblock one or more URLs on FortiGate by removing them from a web filter profile.

**Action:** `fortigate_firewall--unblock_url`

#### Parameters

- `urls` (required): Comma-separated list of URLs to unblock
- `url_filter_profile` (required): Name of the web filter profile to update
- `vdom` (optional): Virtual domain name to target
- `config_name` (optional): Configuration profile name (default: "default")

#### Returns

```json
{
  "not_exist": [],
  "newly_unblocked": ["social-media.com/path"],
  "not_unblock": []
}
```

---

**Back to:** [Integrations Overview](../)

**Last Updated:** March 2026
