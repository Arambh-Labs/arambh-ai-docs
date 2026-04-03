---
layout: default
title: Deployment Guide
permalink: /docs/deployment-guide/
parent: Documentation
nav_order: 1
---

# Deployment Guide

This guide will walk you through the deployment process for Arambh AI.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Deployment Options](#deployment-options)
- [FAQ](#faq)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### System Requirements

#### POC Environment
- **CPU**: 4 cores
- **RAM**: 16 GB
- **Disk Space**: 200 GB
- **Operating System**: Redhat/Rocky Linux 9

#### Production Environment
- **CPU**: 16 cores
- **RAM**: 64 GB
- **Disk Space**: 1 TB
- **Operating System**: Redhat/Rocky Linux 9

### Additional Requirements
- Network access to port 443 (HTTPS)
- Valid SSL/TLS certificates
- Access credentials for SIEM, SOAR, and other security tools
- Azure subscription (for cloud deployment option)

## Installation

Installation steps vary depending on your chosen deployment option (see Deployment Options below).

### For VM-Based Deployment (Customer Network)

#### Step 1: Prepare the Virtual Machine
Ensure your VM meets the system requirements specified in Prerequisites and is running Redhat/Rocky Linux 9.

#### Step 2: Configure Network Security
```bash
# Open port 443 for HTTPS traffic
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```

#### Step 3: Install Dependencies
Follow the installation script provided by Arambh Labs support team.

### For Cloud Deployment (Hosted by Arambh Labs)
Arambh Labs will handle the installation and deployment in your nearest supported Azure region. You will need to provide:
- Network connectivity requirements
- IP whitelist for access control
- Integration details for SIEM, SOAR, and other security tools

## Configuration

### Security Tool Integrations
Configure connections to your existing security infrastructure:
- **SIEM & SOAR**: For ingestings alerts/incidents and querying events
- **EDR (Endpoint Detection & Response)**: For ingestings alerts/incidents and querying endpoint logs
- **Firewall**: For remediation, such as updating policies
- **Identity Provider**: For user context lookups, remediation actions, etc

### Notification Channels
Such as
- **Email**: SMTP configuration for email notifications
- **Microsoft Teams/Slack**: Webhook URLs for Teams integration

### Encryption and Data Security
- All credentials are encrypted at rest
- TLS 1.3 encryption for data in transit
- Optional volume encryption for production environments

## Deployment Options

### Option 1: Hosted by Arambh Labs (Recommended)
Arambh AI is deployed and managed by Arambh Labs in your nearest supported Azure region.

**Benefits:**
- Fully managed infrastructure
- Automatic updates and maintenance
- Enterprise-grade security with Azure Firewall and DDoS protection
- Scalable resources based on your needs
- 24/7 monitoring and support

## Security Features

### Network Security
- **IP Whitelisting**: Restrict access to specific IP addresses
- **Site-to-Site Connectivity**: VPN or direct connect support
- **Cloud Security**: Azure Firewall and DDoS protection (production deployments)

**Architecture:**

![Arambh AI Deployment Architecture]({{ site.baseurl }}/assets/images/deployment-architecture.png)

### Option 2: VM-Based Deployment (Customer Network)
Arambh AI is deployed as a virtual machine within your own network infrastructure.

**Benefits:**
- Complete control over infrastructure
- Data remains within your network
- Customizable network policies
- Direct integration with on-premises systems

**Requirements:**
- Provision VM according to system requirements given above
- Configure network access to port 443
- Manage security and updates
- Provide backup and disaster recovery

### AI Model Data Privacy
In both the hosting options mentioned above, Arambh AI uses models through Microsoft's Azure AI Foundry service.
You may opt to use models from Arambh AI cloud, or from your own Azure subscription.

Azure AI Foundry provides the following data protection guarantees:
- are NOT available to other customers.
- are NOT available to OpenAI or other Azure Direct Model providers.
- are NOT used by Azure Direct Model providers to improve their models or services.
- are NOT used to train any generative AI foundation models without your permission or instruction.
- Customer Data, Prompts, and Completions are NOT used to improve Microsoft or third-party products or services without your explicit permission or instruction.

**Azure AI Foundry** is a fully managed Azure service, hosted by Microsoft, ensuring enterprise-grade security and compliance.

For more information, see: [Microsoft Learn - Data Privacy with Azure OpenAI](https://learn.microsoft.com/en-us/azure/ai-foundry/responsible-ai/openai/data-privacy?tabs=azure-portal)


---

## FAQ

TBD

## Troubleshooting

TBD

### Need Help?
If you're experiencing issues not covered here:
- Contact Arambh Labs support at **support@arambhlabs.com**
- For cloud-hosted deployments, our team provides 24/7 monitoring and support
- Include relevant logs and error messages when reaching out for faster resolution
