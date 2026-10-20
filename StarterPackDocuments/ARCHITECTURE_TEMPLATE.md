# Architecture Document — [Project Title]

**Course:** BACT7602 Cloud Computing — Whitecliffe  
**Version:** <!-- e.g. 1.0 — increment each time you update this document -->  
**Last updated:** <!-- YYYY-MM-DD -->  
**Authors:** <!-- All team member names -->

---

## Table of Contents

1. [Solution Overview](#1-solution-overview)
2. [Architecture Diagram](#2-architecture-diagram)
3. [Component Inventory](#3-component-inventory)
4. [Network Topology](#4-network-topology)
5. [Security Architecture](#5-security-architecture)
6. [Reliability and Availability Design](#6-reliability-and-availability-design)
7. [Cost Design](#7-cost-design)
8. [Infrastructure as Code Approach](#8-infrastructure-as-code-approach)
9. [Architecture Decision Records](#9-architecture-decision-records)
10. [Well-Architected Framework Assessment](#10-well-architected-framework-assessment)

---

## 1. Solution Overview

<!-- Describe the business problem and what your cloud solution does. 
     Aim for 2–3 paragraphs. Answer:
     - What does this system do?
     - Who are its users?
     - What are the key non-functional requirements (scale, uptime, data residency)?
-->

**Business context:**  
<!-- e.g. "A New Zealand logistics company needs a cloud-based order management 
     system capable of handling 500 concurrent users with 99.9% uptime." -->

**Solution summary:**  
<!-- e.g. "A three-tier web application deployed on Azure using App Service for 
     the presentation and application layers, Azure SQL Database for data, and 
     Azure CDN for static asset delivery." -->

**Primary cloud provider:** Microsoft Azure  
**Azure region:** <!-- e.g. australiaeast (Sydney) — justify the region choice below -->  
**Region justification:** <!-- e.g. "australiaeast selected for low latency to NZ users (~25ms), 
     NZ data residency compliance, and availability of required services including 
     Azure SQL Hyperscale." -->

---

## 2. Architecture Diagram

<!-- INSTRUCTIONS FOR CREATING YOUR DIAGRAM:
     
     Recommended free tools:
     - draw.io / diagrams.net (free, exports PNG/SVG, Azure icon library built-in)
       → Go to diagrams.net → Extras → Edit Diagram → use Azure shape library
     - Lucidchart (free education plan via your student email)
     - Azure Architecture Center visio templates (download from Microsoft)
     
     Your diagram MUST show:
     ✓ All Azure services used (use official Azure icons)
     ✓ Network boundaries (virtual networks, subnets)
     ✓ Data flow arrows with labels
     ✓ User entry points (internet, VPN, etc.)
     ✓ External dependencies (if any)
     
     Save as PNG (min 1200px wide) into /docs/images/ and embed below.
     Also keep the source file (.drawio or .lucid) in /docs/images/ so it can be edited.
-->

![Architecture Diagram](./images/architecture-diagram.png)
<!-- Replace the above path once you have created your diagram. -->

**Diagram notes:**  
<!-- Explain any abbreviations or symbols used in the diagram. -->

---

## 3. Component Inventory

<!-- List every Azure resource your solution provisions. 
     Be specific — include the SKU/tier, not just the service name. -->

### 3.1 Compute

| Component | Azure Service | SKU / Tier | Purpose |
|-----------|--------------|------------|---------|
| <!-- e.g. Web front-end --> | <!-- App Service --> | <!-- B2 Standard --> | <!-- Hosts the React web application --> |
| <!-- e.g. API layer --> | | | |
| <!-- e.g. Background processor --> | | | |

### 3.2 Data

| Component | Azure Service | SKU / Tier | Redundancy | Purpose |
|-----------|--------------|------------|------------|---------|
| <!-- e.g. Application database --> | <!-- Azure SQL Database --> | <!-- General Purpose, 2 vCores --> | <!-- Zone-redundant --> | <!-- Stores order and user data --> |
| <!-- e.g. File storage --> | | | | |

### 3.3 Networking

| Component | Azure Service | Configuration | Purpose |
|-----------|--------------|--------------|---------|
| <!-- e.g. Application network --> | <!-- Virtual Network --> | <!-- 10.0.0.0/16, 2 subnets --> | <!-- Isolates app and data tiers --> |
| <!-- e.g. Traffic management --> | | | |
| <!-- e.g. Firewall/WAF --> | | | |

### 3.4 Security and Identity

| Component | Azure Service | Configuration | Purpose |
|-----------|--------------|--------------|---------|
| <!-- e.g. Secret storage --> | <!-- Azure Key Vault --> | <!-- Soft-delete enabled, RBAC model --> | <!-- Stores connection strings and API keys --> |
| <!-- e.g. App identity --> | <!-- Managed Identity --> | <!-- System-assigned on App Service --> | <!-- Grants app access to Key Vault without credentials --> |
| <!-- e.g. Access management --> | | | |

### 3.5 Monitoring and Operations

| Component | Azure Service | Configuration | Purpose |
|-----------|--------------|--------------|---------|
| <!-- e.g. Centralised logs --> | <!-- Log Analytics Workspace --> | <!-- 30-day retention --> | <!-- Aggregates logs from all resources --> |
| <!-- e.g. Application monitoring --> | <!-- Application Insights --> | <!-- Connected to Log Analytics --> | <!-- Tracks requests, failures, response times --> |
| <!-- e.g. Alerts --> | <!-- Azure Monitor Alert Rules --> | <!-- 3 rules (CPU, error rate, availability) --> | <!-- Notifies team of issues --> |

---

## 4. Network Topology

<!-- Describe how your virtual network is segmented and why.
     Reference your diagram for the visual layout. -->

### 4.1 Virtual Network Design

| Virtual Network | Address Space | Location |
|----------------|--------------|---------|
| <!-- vnet-project-dev --> | <!-- 10.0.0.0/16 --> | <!-- australiaeast --> |

| Subnet | CIDR | Purpose | Services deployed |
|--------|------|---------|------------------|
| <!-- snet-app --> | <!-- 10.0.1.0/24 --> | <!-- Application tier --> | <!-- App Service integration, Container Apps --> |
| <!-- snet-data --> | <!-- 10.0.2.0/24 --> | <!-- Data tier --> | <!-- SQL private endpoint, Storage private endpoint --> |
| <!-- snet-mgmt --> | <!-- 10.0.3.0/24 --> | <!-- Management --> | <!-- Bastion Host, jump server --> |

### 4.2 Network Security

<!-- Describe your network security group rules. 
     Table format: list the rules that matter (not the defaults). -->

| Rule | Direction | Priority | Source | Destination | Protocol | Action | Reason |
|------|-----------|----------|--------|-------------|----------|--------|--------|
| <!-- Allow HTTPS inbound --> | Inbound | 100 | Internet | App subnet | TCP 443 | Allow | <!-- Public web traffic --> |
| <!-- Deny direct DB access --> | Inbound | 200 | Internet | Data subnet | Any | Deny | <!-- No direct public access to database --> |
| | | | | | | | |

### 4.3 Private Endpoints

<!-- List any private endpoints you have configured. -->

| Service | Private Endpoint | Private DNS Zone | Justification |
|---------|-----------------|-----------------|--------------|
| <!-- Azure SQL --> | <!-- Yes --> | <!-- privatelink.database.windows.net --> | <!-- Prevents database traffic from traversing public internet --> |
| | | | |

---

## 5. Security Architecture

<!-- This section maps to LO3 and is assessed. 
     Be specific — generic statements score poorly. -->

### 5.1 Identity and Access Management

**Authentication model:**  
<!-- e.g. "Application uses Entra ID (Azure AD) for user authentication via 
     OAuth 2.0 / OpenID Connect. Service-to-service authentication uses 
     system-assigned managed identities — no passwords or client secrets 
     are stored in code or configuration." -->

**Role assignments:**

| Principal | Role | Scope | Justification |
|-----------|------|-------|--------------|
| <!-- App Service Managed Identity --> | <!-- Key Vault Secrets User --> | <!-- Key Vault resource --> | <!-- Minimum privilege: read-only access to secrets --> |
| <!-- DevOps Pipeline Service Principal --> | <!-- Contributor --> | <!-- Resource group --> | <!-- Required to create/update resources during deployment --> |
| <!-- Developer team group --> | <!-- Reader --> | <!-- Subscription --> | <!-- Allows visibility without write access to production --> |

### 5.2 Secrets Management

<!-- Describe how all secrets are handled. -->

- All connection strings, API keys, and passwords are stored in **Azure Key Vault**.
- Application retrieves secrets at runtime using its managed identity — no secrets in code or environment variables.
- Key Vault is configured with: soft-delete enabled (90-day retention), purge protection on, private endpoint (no public network access).

### 5.3 Data Protection

| Data category | At-rest encryption | In-transit encryption | Backup |
|--------------|-------------------|----------------------|--------|
| <!-- Database --> | <!-- Azure SQL default (AES-256) --> | <!-- TLS 1.2 enforced --> | <!-- Automated 7-day point-in-time restore --> |
| <!-- File storage --> | <!-- Storage service encryption (SSE) --> | <!-- HTTPS only (secure transfer required) --> | <!-- Geo-redundant storage --> |

---

## 6. Reliability and Availability Design

### 6.1 Service Level Objective

<!-- Define your target uptime and recovery objectives. -->

| Metric | Target | Justification |
|--------|--------|--------------|
| Availability (uptime) | <!-- e.g. 99.9% (~8.7 hrs downtime/year) --> | <!-- Business requirement; Azure App Service SLA is 99.95% --> |
| Recovery Time Objective (RTO) | <!-- e.g. 4 hours --> | <!-- Time to restore service after a failure --> |
| Recovery Point Objective (RPO) | <!-- e.g. 1 hour --> | <!-- Maximum data loss acceptable --> |

### 6.2 Availability Zone Configuration

<!-- Describe which services are zone-redundant and which are not, and why. -->

| Service | Zone-redundant? | Notes |
|---------|----------------|-------|
| <!-- App Service Plan --> | <!-- Yes (Premium P1v3) --> | <!-- Required for zone redundancy; added cost justified by SLO --> |
| <!-- Azure SQL --> | <!-- Yes --> | <!-- Zone-redundant replica included in General Purpose tier --> |
| <!-- Storage Account --> | <!-- ZRS --> | <!-- Zone-redundant storage; data replicated across 3 zones --> |

### 6.3 Autoscaling Rules

<!-- Describe your scale-out and scale-in rules. -->

**App Service autoscale:**

| Rule | Metric | Threshold | Action | Cool-down |
|------|--------|-----------|--------|-----------|
| Scale out | CPU percentage | > 70% for 5 min | Add 1 instance | 5 minutes |
| Scale in | CPU percentage | < 30% for 10 min | Remove 1 instance | 10 minutes |
| Minimum instances | — | — | 2 | — |
| Maximum instances | — | — | 10 | — |

### 6.4 Backup and Restore

| Resource | Backup method | Frequency | Retention | Recovery tested? |
|----------|--------------|-----------|-----------|-----------------|
| <!-- Azure SQL --> | <!-- Automated point-in-time --> | <!-- Every 5–10 min (log backups) --> | <!-- 7 days --> | <!-- Yes — see Runbook § 4 --> |
| <!-- Blob Storage --> | <!-- Blob versioning + soft delete --> | <!-- Continuous --> | <!-- 7 days --> | <!-- Yes --> |

---

## 7. Cost Design

### 7.1 Estimated Monthly Cost (Development Environment)

<!-- Use the Azure Pricing Calculator (https://azure.microsoft.com/en-us/pricing/calculator/)
     and paste a summary here. Include a link to your saved estimate. -->

| Service | SKU | Estimated NZD/month |
|---------|-----|---------------------|
| | | |
| | | |
| | | |
| **Total** | | **NZD X.XX** |

[View full pricing calculator estimate →](<!-- Paste shareable link from Azure Pricing Calculator -->)

### 7.2 Cost Optimisation Decisions

<!-- List specific choices you made to reduce cost without compromising the design. -->

| Decision | Cost impact | Trade-off |
|----------|-------------|-----------|
| <!-- Used B2 App Service for dev instead of P1v3 --> | <!-- ~40% reduction --> | <!-- No autoscaling in dev; acceptable for testing --> |
| <!-- Scheduled dev environment shutdown (6pm–8am) --> | <!-- ~60% reduction on compute --> | <!-- Manual start required for after-hours work --> |
| <!-- Reserved capacity for SQL (1-year) --> | <!-- ~35% vs pay-as-you-go --> | <!-- Committed spend; acceptable for a stable workload --> |

---

## 8. Infrastructure as Code Approach

### 8.1 Tool Choice

**IaC tool:** Bicep (Azure Resource Manager)  
**Justification:** <!-- e.g. "Bicep was selected as it is the recommended IaC 
     language for Azure deployments, has native IDE support, and produces 
     clean declarative templates with no state file management." -->

### 8.2 Template Structure

```
infra/
├── main.bicep               # Orchestrates all modules; sets scope and parameters
├── modules/
│   ├── appservice.bicep     # App Service plan + web app
│   ├── database.bicep       # Azure SQL server + database
│   ├── network.bicep        # Virtual network, subnets, NSGs
│   ├── keyvault.bicep       # Key Vault + access policies
│   └── monitoring.bicep     # Log Analytics + Application Insights + alerts
└── parameters/
    ├── dev.parameters.json  # Development environment values
    └── prod.parameters.json # Production environment values
```

### 8.3 Idempotency and What-If Validation

<!-- Explain how you ensure templates can be re-run safely. -->

All Bicep templates use `existing` resource references where appropriate and are designed to be idempotent — re-running a deployment produces the same outcome without duplication or errors.

Before each deployment, the pipeline runs:
```bash
az deployment group what-if \
  --resource-group "<rg-name>" \
  --template-file infra/main.bicep \
  --parameters infra/parameters/dev.parameters.json
```

---

## 9. Architecture Decision Records

<!-- An Architecture Decision Record (ADR) documents WHY a major decision was made.
     This is important for assessment — markers want to see your reasoning, not just your outcome.
     Write one ADR for each significant design choice (aim for at least 3–5). -->

### ADR-001: [Short title of decision]

**Date:** <!-- YYYY-MM-DD -->  
**Status:** Accepted  

**Context:**  
<!-- What situation or problem made this decision necessary?
     e.g. "We needed to choose between Azure App Service and Azure Container Apps 
     for hosting our web application." -->

**Decision:**  
<!-- What did you decide?
     e.g. "We selected Azure App Service." -->

**Reasons:**  
<!-- Why did you make this choice? List specific evidence.
     e.g. "App Service provides built-in autoscaling, deployment slots for 
     zero-downtime releases, and direct integration with Azure DevOps. 
     Container Apps was considered but requires containerising the application, 
     adding build complexity out of scope for this project." -->

**Consequences:**  
<!-- What does this decision mean going forward? Any trade-offs?
     e.g. "App Service ties us to Microsoft's managed runtime. We accept this 
     trade-off given the Azure-first strategy." -->

---

### ADR-002: [Short title of decision]

**Date:**  
**Status:** Accepted  

**Context:**  

**Decision:**  

**Reasons:**  

**Consequences:**  

---

### ADR-003: [Short title of decision]

**Date:**  
**Status:** Accepted  

**Context:**  

**Decision:**  

**Reasons:**  

**Consequences:**  

---

<!-- Add more ADRs as needed. Each major design choice should have one. -->

---

## 10. Well-Architected Framework Assessment

<!-- The Azure Well-Architected Framework defines five pillars of a good cloud architecture.
     Rate your solution against each pillar and explain your reasoning.
     Honest self-assessment scores better than inflated claims. -->

| Pillar | Self-rating (1–5) | Key strengths | Known gaps |
|--------|------------------|--------------|-----------|
| **Reliability** | <!-- 1–5 --> | <!-- e.g. Zone-redundant SQL, autoscaling, automated backups --> | <!-- e.g. No multi-region failover in dev environment --> |
| **Security** | <!-- 1–5 --> | <!-- e.g. Managed identities, private endpoints, Key Vault --> | <!-- e.g. Web Application Firewall not implemented (cost) --> |
| **Cost Optimisation** | <!-- 1–5 --> | <!-- e.g. Dev environment uses lower SKUs, scheduled shutdown --> | <!-- e.g. No reserved instances purchased yet --> |
| **Operational Excellence** | <!-- 1–5 --> | <!-- e.g. Full CI/CD pipeline, runbook documented, alerts configured --> | <!-- e.g. No chaos engineering / failure injection testing --> |
| **Performance Efficiency** | <!-- 1–5 --> | <!-- e.g. CDN for static assets, connection pooling on SQL --> | <!-- e.g. Load testing not completed --> |

**Overall architecture maturity:** <!-- e.g. "Production-ready for development/UAT; 
     would require Web Application Firewall and multi-region strategy before 
     full production at scale." -->
