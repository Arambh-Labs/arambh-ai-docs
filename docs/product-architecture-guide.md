---
layout: default
title: Architecture Guide
permalink: /docs/product-architecture-guide/
parent: Documentation
nav_order: 3
---

# Arambh AI — Model & System Documentation
 
Arambh AI is an agentic Security Operations Centre (SOC) assistant that autonomously investigates security alerts and threat-hunt hypotheses. It integrates with SIEM platforms (Google SecOps, Microsoft Sentinel, IBM QRadar, Splunk, Securonix, Elasticsearch), endpoint tools (CrowdStrike Falcon, Cortex XSIAM), threat intelligence sources (AbuseIPDB, APIVoid), and SOAR platforms (Google SecOps SOAR, FortiSOAR).
 
---
 
## 1. Model Planning, Design & Development
 
### Planning Principles
 
- **Security-first design:** All user-supplied input passes through prompt-injection guardrails before reaching any model call.
- **Human-in-the-loop:** Investigations run in either *auto* or *assisted* mode. In assisted mode, the system pauses for analyst approval at designated steps.
- **Modularity:** Tools, workplans, and integrations are independently versioned and hot-swappable without redeploying the core system.
- **Observability:** Every tool call and state transition is logged and stored for audit.
### Investigation Plan Development
 
1. Training content is prepared for each supported threat category — endpoint, cloud, SaaS/Kubernetes, identity, and others — enabling the system to consistently derive appropriate investigation steps for any new alert of that type.
2. Training content is prepared per target query language (SIEM, data lake, or endpoint platform), covering grammar, syntax rules, and known pitfalls so the system generates valid, optimised queries for each environment.
3. Training content is provided per integrated platform to guide environment discovery — how to identify available indexes, enabled log sources, and other contextual signals — allowing the system to adapt its approach to each customer's specific data landscape.
4. Security analysts can optionally supply Standard Operating Procedures (SOPs) to encode their organisation's best practices, escalation policies, and investigative preferences, which the system incorporates alongside the general training content.
5. When a new alert arrives, the system reasons across all of the above — threat category knowledge, query language rules, environment context, and any applicable SOPs — to plan and execute a tailored set of investigation steps.
### LLM Configuration
 
The system uses an ensemble of models specialised for different tasks:
 
| Role | Purpose |
|---|---|
| Fast LLM | Low-latency tasks such as summarisation |
| Smart LLM | Complex reasoning and multi-step decision making |
| Reasoning LLM | High-effort tasks such as field selection and query construction from large fact sets |
 
---
 
## 2. Incident Management
 
### Investigation Lifecycle
 
| Phase | Description |
|---|---|
| Initiated | Alert received; workplan retrieved or generated; investigation record created. |
| Running | System steps through the workplan, dispatching each step to the execution layer. |
| Polling | Long-running tool calls are offloaded asynchronously; investigation pauses until the result is available. |
| Waiting for Input | An approval step is reached; the analyst is notified and the investigation suspends. |
| Completed | All workplan steps executed; verdict generated; summary posted back to the originating platform. |
| Failed | Unrecoverable error or retry threshold exceeded; incident forwarded via configured notification channel. |
 
### Notification Channels
 
- **Platform forwarder:** Investigation analysis is automatically posted as a comment back to the SIEM or SOAR platform the alert originated from.
- **Email forwarder:** Sends failure reports including a direct investigation URL.
- **Jira forwarder:** Creates a Jira issue for failed investigations when configured.
- **Event bus:** An investigation-completed event is published on every completion (success or failure), enabling downstream consumers to react in real time.
---
 
## 3. Core Algorithms
 
**Tool Selection** uses a two-stage semantic search — task-to-agent routing followed by semantic retrieval — with LLM-guided final selection and argument construction. Each selection goes through review cycles to prevent hallucinations and ensure consistency.
 
**Query Generation** applies a multi-stage validation pipeline: field retrieval, LLM query generation, syntax checking, logical consistency checking, and iterative refinement.
 
**Verdict Generation** applies type-specific chain-of-thought reasoning to synthesise all investigation findings into a structured verdict:
 
| Investigation Type | Possible Verdicts |
|---|---|
| Threat Hunt | Proven / Disproven / Inconclusive / Under Observation |
| Phishing | Phishing / Likely Phishing / Spam / Legitimate / Incomplete |
| General | True Positive / Escalation / False Positive / Inconclusive / Under Observation |
 
Verdicts follow a strict precedence ladder based on the proportion of critical investigation steps that completed successfully.
 
---
 
## 4. Output Quality
 
- **Query confidence scores** are computed and attached to every generated query, giving analysts a machine-generated reliability signal.
- **Context budgets** are enforced to prevent context-window overflow that could degrade output quality.
- **Trajectory evaluation** records ground-truth and model-predicted tool-call sequences for offline accuracy assessment.
- **Analyst feedback** on investigation outputs is captured and used to continuously improve workplan retrieval and tool selection.
---
 
## 5. Bias Considerations
 
| Bias Source | Description | Mitigation |
|---|---|---|
| Training data recency | Model knowledge cut-off may lag emerging threat actor TTPs. | Web-search steps in workplans fetch live threat intelligence. |
| Verdict anchoring | Early positive signals may anchor the model toward a more severe verdict even when later findings disprove it. | Strict verdict precedence ladder and critical coverage scoring enforced in every verdict prompt. |
| Tenant data bias | A tenant with few historical investigations may receive lower-quality workplan matches. | A global fallback workplan pool is used when tenant-specific data is sparse. |
| Prompt injection | Maliciously crafted alert payloads may attempt to manipulate model behaviour. | A prompt-injection guardrail is applied to all model inputs; alert data is isolated from system instructions. |
 
Bias monitoring recommendations include tracking verdict distribution (True Positive / False Positive / Inconclusive ratios) per alert type and tenant, and measuring analyst override rates to identify alert categories where model performance needs improvement.
 
---
 
## 6. Data Collection & Privacy
 
No data is collected from a customer environment or used for model training. Organisations may opt in to share aggregated, anonymised statistics — such as failure or escalation rates — to help improve the product.

---
 
## 7. Compliance & Responsible AI — Azure AI Foundry
 
Arambh AI's models are hosted on **Microsoft Azure AI Foundry**, which provides an enterprise-grade compliance and responsible AI framework. The following summarises the relevant certifications and controls that apply.
 
### Responsible AI Principles
 
Microsoft Foundry is built on the [Microsoft Responsible AI Standard](https://www.microsoft.com/ai/responsible-ai), which requires AI systems to be evaluated and governed across three stages:
 
- **Discover** — risks are identified and measured before and after deployment, including adversarial prompt testing for jailbreak vulnerabilities.
- **Protect** — content filters, prompt shields, and abuse detection are applied at the model layer to block harmful outputs before they reach users.
- **Govern** — tracing, monitoring, and compliance integrations provide continuous oversight of model behaviour in production.
Arambh AI's own prompt-injection guardrail layer operates on top of these platform-level protections, providing defence-in-depth.
 
### References
 
- [Responsible AI for Microsoft Foundry](https://learn.microsoft.com/en-us/azure/foundry/responsible-use-of-ai-overview)
- [Manage compliance and security in Microsoft Foundry](https://learn.microsoft.com/en-us/azure/foundry/control-plane/how-to-manage-compliance-security)
- [Microsoft Responsible AI Standard](https://www.microsoft.com/ai/responsible-ai)

## 8. Security Monitoring & Attack Detection
 
The Arambh AI SaaS platform is protected by cloud security services including firewalls and infrastructure-level controls. All infrastructure alerts are monitored by the Arambh AI security operations team.
 
At the AI layer, Arambh AI's built-in guardrails detect and block prompt-injection attempts — where maliciously crafted inputs attempt to manipulate model behaviour — and raise alerts when such attempts are identified.