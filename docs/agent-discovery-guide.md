---
layout: default
title: Agent Discovery Guide
permalink: /docs/agent-discovery-guide/
parent: Documentation
nav_order: 2
has_children: true
---

# Agent Discovery Guide

Learn how to connect Arambh AI with third-party services and platforms to discover different AI agents and their potential blast radius.

---

## Integration Categories

### 🔒 Security & SIEM

Security Information and Event Management platforms for threat detection and analysis, Eg, Google Chronicle, Cisco Splunk, IBM QRadar, etc

### 🔥 Network Security

Firewall and network security platforms for traffic control and threat prevention. Eg, Fortinet Fortigate

### 🛡️ Endpoint Detection & Response

## Supported discovery integrations

| Integration                                                                     | Toolkit name     | What it primarily discovers |
| ------------------------------------------------------------------------------- | ---------------- | --------------------------- | ------------------------------------------------------------------- |
| [Microsoft 365]({{ '/docs/agent-discovery-guide/microsoft-365/'                 | relative_url }}) | `microsoft_office365`       | Entra service principals, OAuth grants, Microsoft 365 data surfaces |
| [Google Workspace]({{ '/docs/agent-discovery-guide/google-workspace/'           | relative_url }}) | `google_workspace`          | Third-party OAuth apps and scopes across Workspace users            |
| [GitHub]({{ '/docs/agent-discovery-guide/github/'                               | relative_url }}) | `github`                    | GitHub Apps, SSO PATs, repos, workflows, org secrets                |
| [Google Cloud Platform]({{ '/docs/agent-discovery-guide/google-cloud-platform/' | relative_url }}) | `gcp`                       | Project IAM, service accounts, keys, cloud surfaces                 |
| [Okta]({{ '/docs/agent-discovery-guide/okta/'                                   | relative_url }}) | `okta`                      | OIDC apps, authorization servers, granted scopes                    |
| [Slack]({{ '/docs/agent-discovery-guide/slack/'                                 | relative_url }}) | `slack`                     | Workspace apps and OAuth scope history                              |
| [LiteLLM Proxy]({{ '/docs/agent-discovery-guide/litellm-proxy/'                 | relative_url }}) | `litellm_proxy`             | Virtual keys, users, teams, models, MCP servers                     |
| [Endpoint Discovery]({{ '/docs/agent-discovery-guide/endpoint-discovery/'       | relative_url }}) | `endpoint_discovery`        | Local AI harnesses, MCP servers, endpoint credentials               |

---

## How to integrate (shared flow)

1. **Choose a toolkit** from the table above and follow its setup page.
2. **Create credentials** with the listed permissions or scopes (read-only where possible).
3. **Add a configuration** in Arambh AI → Integrations → [toolkit] → Configurations.
4. **Run a healthcheck** from the integration page to confirm connectivity.
5. **Run blast radius discovery** (see API below).
6. **Review results** in Agents (agents list and blast-radius graph).

You can run several integrations; each discovery is stored per toolkit (and config). Successful runs also contribute to a **global identity** overlay that links identities across toolkits.

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

## Yes, you can add multiple configurations, and give them a unique name.

## Troubleshooting

TDB

## Need More Help?

### Support Channels

- **Email Support:** support@arambhlabs.com
- **Integrations Team:** integrations@arambhlabs.com

---

**Last Updated:** March 2026
