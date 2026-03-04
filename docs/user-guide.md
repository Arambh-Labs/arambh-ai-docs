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
- **Secure Connection**: All communications are encrypted using TLS 1.3

### First Login

After logging in for the first time:
1. Review your integrated security tools (SIEM, SOAR, EDR, etc.)
2. Verify notification channels (Email, Microsoft Teams)
3. Explore the dashboard to understand your security posture
4. Review any existing alerts and investigations

### Quick Start

Arambh AI automatically begins working upon deployment:
1. **Alert Ingestion**: Arambh AI starts ingesting alerts from your SIEM and other security tools
2. **Automated Investigation**: The AI automatically investigates alerts to determine true positives
3. **Threat Hunting**: Proactive threat hunting runs continuously in the background
4. **Notifications**: You'll receive notifications for confirmed threats and completed investigations

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
- Mean Time to Detect (MTTD) improvements
- Mean Time to Respond (MTTR) reduction
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
- **Reports** - Generate compliance and executive reports
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
- **Needs Human Review**: Complex scenarios requiring analyst input
- **Partially Resolved**: Some aspects resolved, others require follow-up

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
- **Data Sources**: Which security tools were queried
- **Findings**: Suspicious activities or matches discovered
- **Risk Score**: AI-calculated risk assessment
- **Context**: Environmental factors and relevant history
- **Recommendations**: Next steps for investigation or remediation

## Attack Paths & Timeline

When Arambh AI confirms a true positive (either through automated investigation or threat hunting), it reconstructs the full attack path with a detailed timeline.

### Attack Path Visualization

The attack path view shows:
- **Entry Point**: How the attacker initially gained access
- **Lateral Movement**: How the attacker moved through your environment
- **Privilege Escalation**: Steps taken to gain higher privileges
- **Data Access**: What sensitive data or systems were accessed
- **Exfiltration**: Any data theft or external communications
- **Persistence Mechanisms**: How the attacker maintained access

### Interactive Timeline

The timeline provides a chronological view of the attack:
- **Time-ordered Events**: All related security events in sequence
- **Affected Assets**: Endpoints, users, and systems involved at each stage
- **MITRE ATT&CK Mapping**: Tactics and techniques used by the attacker
- **Detection Points**: Where your security tools detected activity
- **Blind Spots**: Gaps where the attacker evaded detection

### Using Attack Paths for Response

The attack path visualization enables you to:
1. **Understand Scope**: See the full extent of the compromise
2. **Prioritize Containment**: Identify critical systems to isolate
3. **Eradicate Threats**: Find and remove all attacker footholds
4. **Recover Systems**: Restore affected systems in the correct order
5. **Prevent Recurrence**: Understand how to prevent similar attacks
6. **Generate Reports**: Create executive and technical incident reports

### Export Options

Attack paths and timelines can be exported as:
- PDF reports for executive briefings
- JSON data for integration with ticketing systems
- Visual diagrams for incident documentation
- MITRE ATT&CK Navigator layers

## Detection Health & MITRE ATT&CK

Arambh AI provides continuous visibility into your detection capabilities using the MITRE ATT&CK framework.

### MITRE ATT&CK Coverage Map

The detection health view shows:
- **Covered Techniques**: Which MITRE ATT&CK techniques you can detect
- **Coverage Percentage**: Overall detection coverage across the framework
- **Coverage Quality**: How effective your detections are for each technique
- **Gaps**: Techniques where you have limited or no detection
- **Trend Analysis**: How your coverage improves over time

### Color-Coded Coverage

- **Green**: Strong detection coverage with multiple, high-quality alerts
- **Yellow**: Partial coverage or lower-confidence detections
- **Red**: Little to no detection capability
- **Gray**: Not applicable to your environment

### Detection Recommendations

Based on your coverage gaps, Arambh AI recommends:
- **New Detection Rules**: Suggested SIEM rules to implement
- **Tool Configuration**: How to improve existing security tool settings
- **Integration Opportunities**: Additional tools that would improve coverage
- **Tuning Priorities**: Which detections need improvement first

### Compliance Mapping

Map your detection coverage to compliance frameworks:
- NIST Cybersecurity Framework
- CIS Controls
- PCI DSS
- HIPAA
- SOC 2

## ROI Analytics

Arambh AI quantifies the value it delivers to your security operations through comprehensive ROI metrics.

### Time Savings Metrics

**Alert Triage Time:**
- Average time saved per alert (automated vs. manual investigation)
- Total hours saved per day/week/month
- Analyst capacity freed up for strategic work

**Investigation Speed:**
- Reduction in Mean Time to Detect (MTTD)
- Reduction in Mean Time to Respond (MTTR)
- Faster incident resolution times

### Efficiency Gains

**False Positive Reduction:**
- Percentage of alerts automatically dismissed with high confidence
- Reduction in alert fatigue
- Improved SOC analyst morale and retention

**Coverage Improvement:**
- Increase in detection coverage (MITRE ATT&CK)
- Threats detected that would have been missed
- Reduction in dwell time for advanced threats

### Cost Analysis

The ROI dashboard calculates:
- **Cost per Alert**: Before and after Arambh AI deployment
- **Analyst Productivity**: Increase in alerts handled per analyst
- **Avoided Breach Costs**: Estimated savings from earlier threat detection
- **Tool Optimization**: Better utilization of existing security investments

### Custom ROI Reports

Generate executive reports showing:
- Month-over-month improvements
- Year-over-year comparisons
- Cost-benefit analysis
- Projected ROI for future periods

## Multi-Tenant Management

Arambh AI is inherently multi-tenant, making it ideal for Managed Security Service Providers (MSSPs) and large enterprises managing multiple business units.

### Tenant Isolation

Each tenant has:
- **Complete Data Isolation**: No data sharing between tenants
- **Independent Configuration**: Separate integrations and settings per tenant
- **Custom Branding**: Optional white-labeling for MSSPs
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
4. Configure initial settings and integrations
5. Assign administrators for the tenant
6. Set resource limits and quotas

**Managing Existing Tenants:**
- View tenant health and activity
- Adjust resource allocations
- Enable/disable features per tenant
- Generate per-tenant reports
- Manage billing and subscriptions (for MSSPs)

### Per-Tenant Features

Each tenant can have:
- Independent SIEM, SOAR, and EDR integrations
- Separate notification channels
- Custom detection rules and thresholds
- Unique compliance requirements
- Individual backup and retention policies

### Cross-Tenant Analytics

For MSSP use cases:
- **Threat Intelligence Sharing**: Aggregate threat intel across tenants (anonymized)
- **Benchmark Reports**: Compare tenant metrics against industry averages
- **Best Practices**: Identify successful configurations to replicate
- **Resource Optimization**: Understand utilization patterns across all tenants

## Integrations

Arambh AI integrates with your existing security infrastructure. For detailed integration guides, see the [Integrations Guide](/docs/integrations-guide/).

### Supported Integrations

**SIEM & SOAR:**
- Alert ingestion and enrichment
- Bidirectional communication for response actions
- Query capabilities for context gathering

**EDR (Endpoint Detection & Response):**
- Endpoint telemetry ingestion
- Host-based investigation data
- Remediation actions (isolate, quarantine, etc.)

**Firewall & Network Security:**
- Network flow logs
- Policy updates and blocking actions
- Threat intelligence integration

**Identity & Access Management:**
- User context enrichment
- Authentication logs
- Privilege information

**Threat Intelligence Platforms:**
- IOC feeds
- Threat actor profiles
- Campaign tracking

### Managing Integrations

Navigate to **Integrations** to:
- View connected tools and their status
- Add new integrations
- Test connectivity
- Update credentials
- Configure data ingestion rates
- Set up custom field mappings

## Settings & Preferences

### User Profile

Access your profile settings to configure:
- Name and contact information
- Time zone preferences (affects timestamp display)
- Default dashboard view
- Language preferences
- Notification preferences

### Notification Configuration

**Email Notifications:**
- Critical threats (true positives)
- Investigation completions
- Threat hunt findings
- System alerts and errors
- Daily/weekly summary reports

**Microsoft Teams/Slack:**
- Configure webhook URLs
- Choose alert severity thresholds
- Customize message formatting
- Test notification delivery

**Notification Thresholds:**
- Set minimum severity for notifications (Low, Medium, High, Critical)
- Configure notification frequency (real-time, batched, scheduled)
- Enable/disable specific event types

### System Configuration

**Alert Processing:**
- Set alert ingestion rates
- Configure auto-investigation triggers
- Define severity thresholds
- Set alert retention periods

**Threat Hunting:**
- Schedule recurring threat hunts
- Configure hunt scope and depth
- Set IOC refresh intervals
- Enable/disable specific hunt types

**Data Retention:**
- Investigation history retention period
- Alert data retention
- Log archival policies
- Compliance-driven retention rules

### Multi-Tenant Settings (Admin Only)

For administrators managing multiple tenants:
- Default tenant on login
- Cross-tenant view preferences
- Global vs. per-tenant settings
- Resource allocation controls

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
- Measure improvements in MTTD and MTTR
- Showcase false positive reduction to stakeholders

**Optimize Tool Usage:**
- Use Arambh AI insights to optimize existing security tool configurations
- Identify underutilized tools that could be sunset
- Make data-driven decisions about new tool purchases

### Multi-Tenant Best Practices (for MSSPs)

**Standardize Where Possible:**
- Create standard integration templates
- Use consistent naming conventions across tenants
- Develop baseline configurations for new tenants

**Customize Where Necessary:**
- Tailor detection thresholds to each tenant's risk profile
- Adjust notification preferences per tenant requirements
- Respect tenant-specific compliance needs

**Leverage Cross-Tenant Insights:**
- Use aggregate threat intelligence across tenants
- Share anonymized best practices
- Benchmark tenant performance against peers

### Integration Health

**Monitor Integration Status:**
- Check integration health daily
- Address connectivity issues promptly
- Keep credentials up to date

**Test Regularly:**
- Verify bidirectional communication with SOAR
- Test notification channels monthly
- Validate alert ingestion from all sources

**Keep Tools Updated:**
- Coordinate Arambh AI updates with integrated tool maintenance
- Test integrations after major updates to connected systems
- Maintain integration documentation

---

## FAQ

### How long does an automated investigation take?
Most investigations complete within 2-10 minutes, depending on the complexity of the alert and the number of data sources queried. Complex investigations involving multiple systems may take up to 30 minutes.

### Can I manually trigger an investigation?
Yes, you can manually initiate an investigation for any alert or security event. Navigate to the alert and click "Investigate Now" to start an AI-powered investigation.

### What happens if Arambh AI is unsure about an alert?
If the AI cannot make a high-confidence determination, the investigation will be marked as "Needs Human Review" and routed to your analysts with all the context and evidence collected during the investigation.

### How does threat hunting differ from automated investigation?
Automated investigation responds to existing alerts, while threat hunting proactively searches for threats that may have evaded your detection tools. Think of investigation as reactive and threat hunting as proactive.

### Can I create custom threat hunts?
Yes, you can create custom hunt queries tailored to your environment, specific threats, or compliance requirements. Navigate to Threat Hunts → Create Custom Hunt.

### How often is the MITRE ATT&CK coverage updated?
Your MITRE ATT&CK coverage map is updated in real-time as detections are triggered and as you add or modify integrations. Historical trends are calculated daily.

### What is considered a "true positive" for ROI calculations?
A true positive is an alert that Arambh AI confirms as a genuine security threat requiring action. This includes confirmed malware, unauthorized access, data exfiltration, and other verified malicious activity.

### How does multi-tenancy work for data isolation?
Each tenant has completely isolated data storage and processing. Data from one tenant is never accessible to another tenant. Multi-tenant admins can switch between tenant views but cannot see cross-tenant data without appropriate permissions.

### Can I integrate Arambh AI with my existing SOAR platform?
Yes, Arambh AI integrates bidirectionally with major SOAR platforms. It can both receive alerts from SOAR and send response actions back to SOAR for execution.

### How is my data protected?
All data is encrypted in transit (TLS 1.3) and at rest. When using Azure Direct Models, your data is NOT shared with other customers or used to train AI models. See the [Deployment Guide](/docs/deployment-guide/#security-features) for complete security details.

### Can I export investigation reports for compliance?
Yes, investigation reports, threat hunt findings, and attack paths can be exported as PDF, JSON, or CSV for compliance documentation, executive reporting, or integration with external systems.

### How do I add a new security tool integration?
Navigate to Settings → Integrations → Add Integration. Select your tool type, provide connection details and credentials, test the connection, and save. See the [Integrations Guide](/docs/integrations-guide/) for detailed instructions.

### What should I do with confirmed true positives?
Review the investigation report and attack path, assess the scope of the compromise, initiate your incident response procedures, and use Arambh AI's recommended actions to contain and remediate the threat.

### How do I contact support?
Email support@arambhlabs.com for assistance. For cloud-hosted deployments, 24/7 monitoring and support are included.

---

## Troubleshooting

### Investigations Not Starting Automatically
**Problem:** Alerts are being received but investigations aren't triggering.

**Solution:**
1. Check integration status in Settings → Integrations
2. Verify alert format matches expected schema
3. Check alert severity meets auto-investigation threshold
4. Review investigation queue for backlog
5. Check system logs for errors
6. Verify SIEM connection is active
7. Contact support if problem persists

### Investigation Taking Too Long
**Problem:** Investigation has been running for longer than expected.

**Solution:**
1. Complex investigations can take up to 30 minutes
2. Check if data sources are responding slowly
3. Verify network connectivity to integrated tools
4. Review investigation progress log
5. If stuck for over 1 hour, manually cancel and restart
6. Check for API rate limiting on integrated tools

### False Positive Rate Still High
**Problem:** Too many alerts are still being classified as false positives.

**Solution:**
1. Use Arambh AI insights to tune upstream detection rules
2. Review common false positive patterns in investigation reports
3. Adjust alert severity thresholds in your SIEM
4. Disable or modify overly sensitive detection rules
5. Ensure proper context is available (EDR, logs, etc.)
6. Allow 2-4 weeks for AI to learn your environment

### Threat Hunts Not Finding Anything
**Problem:** Threat hunts complete but show no findings.

**Solution:**
1. This is often good news - it means no threats were found
2. Verify hunt queries are correctly scoped to your environment
3. Check if data sources have sufficient historical data
4. Review hunt parameters and adjust sensitivity
5. Try different hunt types (behavioral vs. IOC-based)
6. Ensure integrated tools are providing complete telemetry

### MITRE ATT&CK Coverage Looks Low
**Problem:** Detection coverage map shows many red/uncovered areas.

**Solution:**
1. This is normal for initial deployments
2. Review Arambh AI's recommendations for new detection rules
3. Prioritize coverage for techniques relevant to your industry
4. Implement detections for critical techniques first
5. Use purple team exercises to validate coverage
6. Coverage naturally improves over time with tuning

### Attack Path Not Showing Complete Timeline
**Problem:** Attack path visualization is missing some events.

**Solution:**
1. Check if all relevant data sources are integrated
2. Verify log retention settings allow historical queries
3. Some events may be below display threshold (adjust filters)
4. Check for gaps in data collection during the attack timeframe
5. Manually add known events to the timeline if needed
6. Review investigation logs for data source query failures

### ROI Metrics Seem Inaccurate
**Problem:** ROI calculations don't match expected values.

**Solution:**
1. Verify baseline metrics were properly configured during setup
2. Check if sufficient time has passed (minimum 2 weeks recommended)
3. Review calculation methodology in ROI settings
4. Manually validate a sample of investigations
5. Adjust baseline assumptions if environment has changed
6. Export raw data for manual verification

### Cannot Switch Between Tenants
**Problem:** Tenant selector not working or not visible.

**Solution:**
1. Verify you have multi-tenant access permissions
2. Check if you're assigned to multiple tenants
3. Clear browser cache and refresh
4. Ensure your account has the required role (Admin or MSP Operator)
5. Contact tenant administrator to verify your access
6. Try logging out and back in

### Integration Status Shows "Disconnected"
**Problem:** Previously working integration is now showing as disconnected.

**Solution:**
1. Test connectivity from Settings → Integrations → Test Connection
2. Check if credentials have expired or been rotated
3. Verify firewall rules still allow outbound connections
4. Check if integrated tool's API is accessible
5. Review API rate limits (may need to adjust query frequency)
6. Update credentials if they've changed
7. Check integrated tool's status page for outages

### Notifications Not Being Received
**Problem:** Not receiving email or Teams notifications for critical alerts.

**Solution:**
1. Verify notification settings in Settings → Notifications
2. Check notification severity threshold settings
3. Confirm email/Teams webhook URLs are correct
4. Test notification channels with manual test message
5. Check spam/junk folders for emails
6. Verify Teams webhook hasn't expired
7. Check notification queue for delivery failures

### Dashboard Metrics Not Updating
**Problem:** Dashboard shows stale or outdated information.

**Solution:**
1. Refresh the page (Ctrl+R or Cmd+R)
2. Check if auto-refresh is enabled (Settings → Dashboard)
3. Verify data sources are actively sending data
4. Check integration health status
5. Clear browser cache if problem persists
6. Try different browser or incognito mode
7. Check for browser console errors (F12)

### Cannot Export Investigation Report
**Problem:** Report export fails or produces blank file.

**Solution:**
1. Check if investigation is fully completed (not still in progress)
2. Verify you have export permissions
3. Try different export format (PDF vs. JSON)
4. Check browser popup blocker settings
5. Ensure sufficient disk space on your device
6. Try exporting a smaller date range
7. Contact support if specific investigation won't export

### Slow Dashboard Performance
**Problem:** Dashboard loads slowly or feels sluggish.

**Solution:**
1. Check internet connection speed
2. Reduce number of active dashboard widgets
3. Adjust dashboard data refresh interval
4. Filter views to shorter time ranges
5. Clear browser cache and cookies
6. Disable browser extensions temporarily
7. Close unnecessary browser tabs
8. Try a different browser or device

### Need Additional Help?

If your issue isn't resolved:
- Contact support at **support@arambhlabs.com**
- For cloud-hosted deployments: 24/7 monitoring and support included
- Include relevant details:
  - Investigation ID (if applicable)
  - Integration type experiencing issues
  - Error messages or screenshots
  - Steps to reproduce the problem
- Check deployment status and logs on your system
