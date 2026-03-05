---
layout: default
title: User Guide
permalink: /docs/user-guide/
parent: Documentation
nav_order: 4
---

# User Guide

Welcome to Arambh AI! This guide will help you get started with our AI-powered Security Operations platform and make the most of all available features.

## Table of Contents
- [Getting Started](#getting-started)
- [Dashboard Overview](#dashboard-overview)
- [Automated Investigation](#automated-investigation)
- [Proactive Threat Hunting](#proactive-threat-hunting)
- [Attack Paths & Timeline](#attack-paths--timeline)
- [Detection Health & MITRE ATT&CK](#detection-health--mitre-attck)
- [ROI Analytics](#roi-analytics)
- [Multi-Tenant Management](#multi-tenant-management)
- [Integrations](#integrations)
- [Best Practices](#best-practices)
- [FAQ](#faq)
- [Troubleshooting](#troubleshooting)

## Getting Started

### Accessing Arambh AI

Once your deployment is complete (see [Deployment Guide](/docs/deployment-guide/)), you can access Arambh AI through:
- **Web Interface**: `https://your-arambh-instance.com` (port 443)

## Dashboard Overview

The Arambh AI dashboard provides a comprehensive view of your security operations and AI-driven insights.

### Key Dashboard Components

#### 1. Security Operations Overview
- **Active Investigations**: Currently running automated investigations
- **Threat Hunts**: Active and completed proactive threat hunting activities
- **True Positives**: Confirmed security incidents requiring attention
- **Alert Volume**: Real-time view of incoming alerts from integrated tools

#### 2. MITRE ATT&CK Coverage
- Visual representation of detection coverage across MITRE ATT&CK framework
- Identify gaps in your security controls
- Track improvements over time
- See which tactics and techniques you can detect

#### 3. ROI Metrics
- Time saved through automated investigations
- Alert reduction statistics (false positive elimination)
- Mean Time to Investigate (MTTI) improvements
- SOC analyst efficiency gains

#### 4. Multi-Tenant View (for MSP/Enterprise)
- Switch between different tenants/organizations
- Aggregate view across all tenants
- Per-tenant security metrics
- Centralized management console

### Navigation

- **Dashboard** - Main security operations overview
- **Investigations** - View automated and manual investigations
- **Threat Hunts** - Access proactive threat hunting results
- **Attack Paths** - Visualize confirmed attack chains
- **Integrations** - Manage connected security tools
- **Workplans** - Workplans runs for various alert types
- **Settings** - Configure system preferences and integrations

## Automated Investigation

Arambh AI's automated investigation engine analyzes security alerts and incidents to determine if they represent true threats, dramatically reducing false positives and analyst workload.

### How Automated Investigation Works

![Autonomous Investigation Flow](/assets/images/autonomousinvestigation.png)

When an alert is received from your SIEM or other security tools, Arambh AI:

1. **Alert Enrichment**: Gathers context from multiple sources (EDR, firewall logs, identity systems, threat intelligence)
2. **AI Analysis**: Uses advanced AI models to analyze the alert in context
3. **Evidence Collection**: Automatically queries relevant security tools for additional telemetry
4. **Threat Correlation**: Correlates events across different security tools and timeframes
5. **Risk Assessment**: Determines the severity and likelihood of a true positive
6. **Investigation Report**: Generates a comprehensive investigation report with findings
7. **Recommended Actions**: Suggests remediation steps based on the analysis

### Investigation Outcomes

Each investigation results in one of the following:
- **True Positive**: Confirmed threat requiring immediate attention
- **False Positive**: Alert dismissed with detailed reasoning
- **Escalation**: Require analyst input and additional context
- **Inconclusive**: Failed to reach a conclusion due to missing/invalid configuration inputs for some tools

### Viewing Investigation Results

Navigate to the **Investigations** page to:
- View all completed and in-progress investigations
- Filter by status, severity, or time range
- Review detailed investigation reports
- Export findings for compliance or reporting
- Take remediation actions directly from the interface

### Investigation Timeline

Each investigation includes a detailed timeline showing:
- When the alert was received
- What data sources were queried
- Key findings at each stage
- Decision points and AI reasoning
- Final determination and recommended actions

## Proactive Threat Hunting

Beyond reactive alert investigation, Arambh AI continuously performs proactive threat hunting to identify potential threats that may have evaded traditional detection.

### Threat Hunting Capabilities

**Behavioral Analysis:**
- Identifies anomalous behavior patterns across your environment
- Detects living-off-the-land (LOTL) techniques
- Uncovers stealthy lateral movement
- Identifies credential misuse and privilege escalation

**IOC Hunting:**
- Searches for Indicators of Compromise (IOCs) from threat intelligence feeds
- Hunts for known malware signatures and behaviors
- Identifies connections to malicious infrastructure
- Correlates multiple weak signals into strong indicators

**Hypothesis-Based Hunting:**
- AI generates and tests threat hypotheses based on your environment
- Explores "what-if" scenarios for emerging threats
- Adapts to new TTPs (Tactics, Techniques, and Procedures)
- Learns from past incidents to improve future hunts

### Viewing Threat Hunt Results

Access the **Threat Hunts** page to:
- View completed and ongoing threat hunts
- Review findings and severity ratings
- See matched IOCs and behavioral patterns
- Investigate suspected threats deeper
- Create custom hunt queries
- Schedule recurring hunts

### Hunt Report Details

Each threat hunt provides:
- **Hunt Hypothesis**: What the AI was looking for and why
- **Findings**: Suspicious activities or matches discovered
- **Risk Score**: AI-calculated risk assessment
- **Recommendations**: Next steps for investigation or remediation

## Attack Paths & Timeline

When Arambh AI confirms a true positive (either through automated investigation or threat hunting), it reconstructs the full attack path with a detailed timeline.

### Interactive Timeline

The timeline provides a chronological view of the attack:
- **Time-ordered Events**: All related security events in sequence
- **Affected Assets**: Endpoints, users, and systems involved at each stage
- **MITRE ATT&CK Mapping**: Tactics and techniques used by the attacker

### Using Attack Paths for Response

The attack path visualization enables you to:
1. **Understand Scope**: See the full extent of the compromise
2. **Prioritize Containment**: Identify critical systems to isolate
3. **Eradicate Threats**: Find and remove all attacker footholds
4. **Recover Systems**: Restore affected systems in the correct order
5. **Prevent Recurrence**: Understand how to prevent similar attacks
6. **Generate Reports**: Create executive and technical incident reports


## Detection Health & MITRE ATT&CK

Arambh AI provides continuous visibility into your detection capabilities using the MITRE ATT&CK framework.

### MITRE ATT&CK Coverage Map

The detection health view shows:
- **Covered Techniques**: Which MITRE ATT&CK techniques you can detect
- **Coverage Percentage**: Overall detection coverage across the framework
- **Coverage Quality**: How effective your detections are for each technique
- **Gaps**: Techniques where you have limited or no detection
- **Trend Analysis**: How your coverage improves over time


### Detection Recommendations

Based on your coverage gaps, Arambh AI recommends:
- **New Detection Rules**: Suggested SIEM rules to implement
- **Tool Configuration**: How to improve existing security tool settings
- **Integration Opportunities**: Additional tools that would improve coverage
- **Tuning Priorities**: Which detections need improvement first


## Multi-Tenant Management

Arambh AI is inherently multi-tenant, making it ideal for Managed Security Service Providers (MSSPs) and large enterprises managing multiple business units.

### Tenant Isolation

Each tenant can have:
- **Independent Configuration**: Separate integrations, workplans, network maps and other settings per tenant
- **Separate User Access**: Role-based access control per tenant

### Centralized Management

From the main console, administrators can:
- **Switch Between Tenants**: Quick tenant selector in the navigation bar
- **Aggregate Dashboard**: View metrics across all tenants
- **Bulk Configuration**: Apply settings to multiple tenants simultaneously
- **Comparative Analytics**: Compare performance across tenants

### Tenant Administration

**Creating a New Tenant:**
1. Navigate to **Admin** → **Tenants**
2. Click **Add New Tenant**
3. Enter tenant details (name, organization, contact)

### Per-Tenant Features

Each tenant can have:
- Independent SIEM, SOAR, and EDR integrations
- Separate notification channels
- Custom detection rules and thresholds
- Unique compliance requirements

## Best Practices

### Getting the Most from Automated Investigation

**Review Investigation Reports:**
- Regularly review completed investigations, even false positives
- Look for patterns in false positive reasoning to improve alert tuning
- Use AI insights to improve your detection rules

**Provide Feedback:**
- Mark investigations as helpful or not helpful
- Add comments to complex investigations for your team
- Report any incorrect classifications to improve AI accuracy

**Tune Alert Sources:**
- Use Arambh AI's findings to tune your SIEM and other tools
- Disable or modify rules that consistently generate false positives
- Create new rules for gaps identified by threat hunting

### Optimizing Threat Hunting

**Schedule Strategic Hunts:**
- Run comprehensive hunts during low-activity periods
- Schedule IOC hunts after threat intelligence updates
- Perform behavioral hunts regularly (weekly or monthly)

**Act on Findings:**
- Don't let hunt results sit unreviewed
- Prioritize high-severity findings for immediate investigation
- Use hunt insights to improve detection coverage

**Customize Hunt Queries:**
- Create custom hunts targeting your specific environment
- Focus on your most critical assets and users
- Hunt for threats relevant to your industry

### Leveraging MITRE ATT&CK Insights

**Close Detection Gaps:**
- Prioritize implementing detections for red-highlighted techniques
- Focus on techniques commonly used in your industry
- Balance coverage across all tactics (initial access, execution, etc.)

**Validate Coverage:**
- Perform purple team exercises to validate detections
- Use Arambh AI's coverage map to track improvement
- Share coverage metrics with leadership

### Maximizing ROI

**Document Time Savings:**
- Track analyst time saved through automation
- Measure improvements in MTTI and MTTR
- Showcase false positive reduction to stakeholders

**Optimize Tool Usage:**
- Use Arambh AI insights to optimize existing security tool configurations
- Identify underutilized tools that could be sunset
- Make data-driven decisions about new tool purchases

---

## FAQ

### How long does an automated investigation take?
Most investigations complete within 2-10 minutes, depending on the complexity of the alert and the number of data sources queried.

### Can I manually trigger an investigation?
Yes, you can manually initiate an investigation for any alert or security event. Navigate to the Investigations page and click "Add New" to start an AI-powered investigation.

### What happens if Arambh AI is unsure about an alert?
If the AI cannot make a high-confidence determination, the investigation will be marked as "Escalation" and routed to your analysts with all the context and evidence collected during the investigation.

### How does threat hunting differ from automated investigation?
Automated investigation responds to existing alerts, while threat hunting proactively searches for threats that may have evaded your detection tools. Think of investigation as reactive and threat hunting as proactive.

### Can I create custom threat hunts?
Yes, you can create custom hunt queries tailored to your environment, specific threats, or compliance requirements. Navigate to Threat Hunts → Add New.

### How often is the MITRE ATT&CK coverage updated?
Your MITRE ATT&CK coverage map is updated in real-time as new alerts get investigated. Historical trends are calculated daily.

### What is considered a "true positive" for ROI calculations?
A true positive is an alert that Arambh AI confirms as a genuine security threat requiring action. This includes confirmed malware, unauthorized access, data exfiltration, and other verified malicious activity.

### Can I integrate Arambh AI with my existing SOAR platform?
Yes, Arambh AI integrates bidirectionally with major SOAR platforms. It can both receive alerts from SOAR and send response actions back to SOAR for execution.

---

## Troubleshooting
TBD

### Need Additional Help?

If your issue isn't resolved:
- Contact support at **support@arambhlabs.com**
- For cloud-hosted deployments: 24/7 monitoring and support included
- Include relevant details:
  - Error messages or screenshots
  - Steps to reproduce the problem
- Check deployment status and logs on your system
