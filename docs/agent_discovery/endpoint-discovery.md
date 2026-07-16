---
layout: default
title: Endpoint Discovery
permalink: /docs/agent-discovery-guide/endpoint-discovery/
parent: Agent Discovery Guide
nav_order: 8
---

# Endpoint Discovery

Connect to a collection server that reports local AI harnesses, MCP servers, credentials, and reachability — then build an endpoint blast-radius graph.

**Toolkit name:** `endpoint_discovery`

---

## Table of Contents

- [Configuration](#configuration)
- [Prerequisites](#prerequisites)
- [Setup instructions](#setup-instructions)

---

## Configuration

| Field                          | Type   | Required | Description                                                     |
| ------------------------------ | ------ | -------- | --------------------------------------------------------------- |
| **Collection server BASE URL** | String | Yes      | Base URL of the endpoint collection server that exposes sensors |

---

## Prerequisites

1. Deploy (or point at) a **collection server** that Arambh AI can reach.
2. Sensors should report at least **agents**; MCP, GitHub tokens, and reach are optional but enrich the graph.
3. Healthcheck calls the agents sensor — that path must be available.

---

## Setup instructions

### 1. Deploy / locate the collection server

Ensure the collection server is running and publishing observations for the endpoints you care about.

### 2. Configure in Arambh AI

1. Integrations → Endpoint Discovery → **Add Configuration**
2. Enter the **Collection server BASE URL**
3. Save and run **Healthcheck**

---

**Last Updated:** July 2026
