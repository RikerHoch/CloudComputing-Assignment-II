# [Web-App Modernisation] — BACT7602 Cloud Computing
<!-- Replace [Project Title] with your project name, e.g. "AcmeRetail Cloud Migration" -->


<!-- Pipeline / deployment status badges — replace URLs after you set up your pipeline -->
![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Azure](https://img.shields.io/badge/platform-Microsoft%20Azure-0078D4?logo=microsoftazure)
![IaC](https://img.shields.io/badge/IaC-Bicep-blue)

---

## Team

| Name    | Student ID | GitHub / ADO Username | Role |
|---------|------------|-----------------------|------|
| Riker   | 20241490 | | Coder |
| Caleb   | 20240970 | | Coder |
| Shatha  | 20251597 | | Documenter |
| Fatimah | 20251554 | | Documenter |

**Course:** BACT7602 Cloud Computing — Whitecliffe  
**Assessment:** Assessment 2 — Group Cloud Project (70%)  
**Submission date:** End of Week 18  
**Repository URL:** <!-- Paste your GitHub or Azure DevOps Repos URL here -->

---

## Project Overview

<!-- Write 3–5 sentences describing what your project does and why it exists.
     Answer: What is the business problem? What cloud solution did you build?
     Example: "AcmeRetail is a New Zealand e-commerce startup requiring a scalable, 
     highly available web platform on Azure. This project provisions the full 
     infrastructure using Bicep templates and deploys via an Azure DevOps pipeline..." -->

**Primary cloud provider:** Microsoft Azure  
**Secondary provider (if applicable):** <!-- AWS / GCP / None -->  
**Azure region:** <!-- e.g. australiaeast -->  
**Target environment:** <!-- e.g. Development + Production -->

---

## Architecture Summary

<!-- One-paragraph summary of your architecture. Full detail is in ARCHITECTURE.md. -->

See [ARCHITECTURE.md](./docs/ARCHITECTURE.md) for the full architecture description and diagram.

**Key Azure services used:**

| Service | Purpose |
|---------|---------|
| <!-- e.g. Azure App Service --> | <!-- e.g. Hosts the web application --> |
| <!-- e.g. Azure SQL Database --> | <!-- e.g. Managed relational database --> |
| <!-- e.g. Azure Key Vault --> | <!-- e.g. Stores secrets and connection strings --> |
| <!-- e.g. Azure Monitor --> | <!-- e.g. Centralised logging and alerting --> |
| | |

---

## Repository Structure

```
/
├── infra/                   # Infrastructure as Code (Bicep templates)
│   ├── main.bicep           # Root module — deploys all resources
│   ├── modules/             # Reusable Bicep modules
│   │   ├── appservice.bicep
│   │   ├── database.bicep
│   │   └── keyvault.bicep
│   └── parameters/
│       ├── dev.parameters.json
│       └── prod.parameters.json
│
├── src/                     # Application source code (if applicable)
│   └── ...
│
├── .pipelines/              # CI/CD pipeline definitions
│   └── azure-pipelines.yml  # OR .github/workflows/deploy.yml
│
├── docs/                    # Project documentation
│   ├── ARCHITECTURE.md      # Architecture description and diagram
│   ├── RUNBOOK.md           # Deployment, rollback, restore, on-call
│   └── CONTRIBUTION_LOG.md  # Weekly per-member contribution records
│
├── tests/                   # Any infrastructure or smoke tests
│
├── FINAL_REPORT.md          # Assessment 2 final report (1,500–2,000 words)
├── AI_DECLARATION.md        # AI tool usage declaration (required)
└── README.md                # This file
```

---

## Prerequisites

Before deploying this project, ensure you have the following installed and configured:

| Tool | Version | Purpose |
|------|---------|---------|
| [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) | ≥ 2.50 | Deploy and manage Azure resources |
| [Bicep CLI](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/install) | ≥ 0.22 | Compile and validate Bicep templates |
| [Git](https://git-scm.com/) | ≥ 2.40 | Version control |
| [VS Code](https://code.visualstudio.com/) + Bicep extension | Latest | Template authoring |

**Required access:**
- An active Azure subscription (Student subscription is sufficient for development)
- Contributor role on the target resource group, OR Owner to create the resource group

---

## Quick Deploy

> **Warning:** Review `parameters/dev.parameters.json` and update all placeholder values before running. Never commit real secrets to this repository.

### Step 1 — Log in to Azure

```bash
az login
az account set --subscription "<your-subscription-id>"
```

### Step 2 — Create the resource group

```bash
az group create \
  --name "rg-<project-name>-dev" \
  --location "australiaeast"
```

### Step 3 — Deploy the infrastructure

```bash
az deployment group create \
  --resource-group "rg-<project-name>-dev" \
  --template-file infra/main.bicep \
  --parameters infra/parameters/dev.parameters.json
```

### Step 4 — Verify deployment

```bash
az resource list \
  --resource-group "rg-<project-name>-dev" \
  --output table
```

For full deployment instructions, rollback steps, and on-call checks, see [RUNBOOK.md](./docs/RUNBOOK.md).

---

## Pipeline

<!-- Describe your CI/CD pipeline briefly. -->

This project uses <!-- Azure DevOps Pipelines / GitHub Actions --> for automated build and deployment.

**Pipeline file:** `.pipelines/azure-pipelines.yml`  
**Triggers:** Push to `main` branch; pull requests to `main`  
**Stages:**
1. **Build** — Validates Bicep templates (`az bicep build --lint`)
2. **Test** — Runs what-if validation against target subscription
3. **Deploy (Dev)** — Deploys to development resource group
4. **Deploy (Prod)** — Manual approval gate required

---

## Environment Variables and Secrets

> **Never store secrets in this repository.** All sensitive values must be stored in Azure Key Vault and referenced by the pipeline via a service connection.

| Variable | Where stored | Description |
|----------|-------------|-------------|
| `SQL_ADMIN_PASSWORD` | Azure Key Vault | Database administrator password |
| `APP_CLIENT_SECRET` | Azure Key Vault | Application service principal secret |
| <!-- add more --> | | |

---

## Contributing

All team members commit directly to feature branches and open pull requests to `main`. At least one other team member must review and approve before merging.

**Branch naming convention:** `feature/<your-name>/<short-description>`  
Example: `feature/jane-smith/add-keyvault-module`

**Commit message format:**
```
<type>: <short description>

Types: feat | fix | docs | infra | test | chore
Example: "feat: add autoscale rules to App Service plan"
```

---

## AI Tool Declaration

<!-- Required for assessment submission. -->

This project used AI tools as follows:

| Tool | How it was used |
|------|----------------|
| <!-- e.g. GitHub Copilot --> | <!-- e.g. Code suggestions for Bicep templates --> |
| <!-- e.g. Claude / ChatGPT --> | <!-- e.g. Reviewed architecture decisions, suggested fixes --> |

All AI-assisted content was reviewed, tested, and validated by the team before inclusion. See `AI_DECLARATION.md` for the full declaration.

---

## Licence

This project is submitted for academic assessment purposes at Whitecliffe. Not for commercial use.
