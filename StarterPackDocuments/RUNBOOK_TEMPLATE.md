# Operations Runbook — [Project Title]

**Course:** BACT7602 Cloud Computing — Whitecliffe  
**Version:** <!-- e.g. 1.0 -->  
**Last updated:** <!-- YYYY-MM-DD -->  
**Authors:** <!-- All team member names -->

> **Purpose:** This runbook describes every operational procedure for this project — how to deploy, validate, roll back, restore from backup, and perform health checks. Anyone on the team should be able to follow these steps without additional guidance.

---

## Table of Contents

1. [Prerequisites and Access](#1-prerequisites-and-access)
2. [Environment Reference](#2-environment-reference)
3. [Deployment Procedure](#3-deployment-procedure)
4. [Post-Deployment Verification (Smoke Tests)](#4-post-deployment-verification-smoke-tests)
5. [Rollback Procedure](#5-rollback-procedure)
6. [Restore from Backup](#6-restore-from-backup)
7. [On-Call Health Checks](#7-on-call-health-checks)
8. [Common Issues and Resolutions](#8-common-issues-and-resolutions)

---

## 1. Prerequisites and Access

### 1.1 Tools Required

| Tool | Version | Install |
|------|---------|---------|
| Azure CLI | ≥ 2.50 | `winget install Microsoft.AzureCLI` or [docs](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) |
| Bicep CLI | ≥ 0.22 | `az bicep install` |
| Git | ≥ 2.40 | [git-scm.com](https://git-scm.com/) |

### 1.2 Required Access

Before running any procedure, confirm you have:

- [ ] Azure account logged in: run `az account show` and verify correct subscription
- [ ] Contributor role on resource group `rg-<project>-<env>`
- [ ] Access to Azure Key Vault: run `az keyvault secret list --vault-name <kv-name>` — should return a list
- [ ] Repository cloned locally and on latest `main`: run `git pull origin main`

### 1.3 Login

```bash
# Log in to Azure (opens browser)
az login

# Confirm the correct subscription is active
az account show --output table

# If wrong subscription, switch:
az account set --subscription "<subscription-name-or-id>"
```

---

## 2. Environment Reference

<!-- Fill in your actual resource names. Keep this table updated. -->

| Parameter | Development | Production |
|-----------|------------|-----------|
| Resource Group | `rg-<project>-dev` | `rg-<project>-prod` |
| Azure Region | `australiaeast` | `australiaeast` |
| App Service name | `app-<project>-dev` | `app-<project>-prod` |
| App Service Plan | `asp-<project>-dev` | `asp-<project>-prod` |
| SQL Server | `sql-<project>-dev` | `sql-<project>-prod` |
| SQL Database | `sqldb-<project>-dev` | `sqldb-<project>-prod` |
| Key Vault | `kv-<project>-dev` | `kv-<project>-prod` |
| Storage Account | `st<project>dev` | `st<project>prod` |
| Log Analytics Workspace | `law-<project>-dev` | `law-<project>-prod` |

**Subscription ID:** `<!-- paste your subscription ID -->`  
**Tenant ID:** `<!-- paste your tenant ID -->`

---

## 3. Deployment Procedure

> **When to use this:** Initial deployment of the environment, or when a full re-deployment is required after infrastructure changes.

> ⚠️ **Production deployments** require review by at least two team members before proceeding.

### Step 1 — Validate templates locally

```bash
# Build and lint all Bicep templates
az bicep build --file infra/main.bicep

# Run what-if to preview changes (ALWAYS do this before deploying to production)
az deployment group what-if \
  --resource-group "rg-<project>-<env>" \
  --template-file infra/main.bicep \
  --parameters infra/parameters/<env>.parameters.json

# Review the output carefully — look for any unexpected deletions or changes
```

**Expected output:** A list of resources to be created/modified. If any unexpected deletions appear, stop and investigate before proceeding.

### Step 2 — Create resource group (first deployment only)

```bash
az group create \
  --name "rg-<project>-<env>" \
  --location "australiaeast" \
  --tags project="<project>" environment="<env>" team="<team-name>"
```

### Step 3 — Deploy infrastructure

```bash
az deployment group create \
  --resource-group "rg-<project>-<env>" \
  --template-file infra/main.bicep \
  --parameters infra/parameters/<env>.parameters.json \
  --name "deploy-$(date +%Y%m%d-%H%M)" \
  --output table
```

**Expected duration:** <!-- e.g. 8–12 minutes for a full deployment -->  

**Expected output:** All resources listed with `Succeeded` state.

### Step 4 — Deploy application code

<!-- If you have application code separate from infrastructure: -->

```bash
# Via Azure DevOps / GitHub Actions pipeline:
# Push to main branch triggers automatic deployment.

# Manual deployment (if pipeline is unavailable):
az webapp deploy \
  --resource-group "rg-<project>-<env>" \
  --name "app-<project>-<env>" \
  --src-path "./dist/app.zip" \
  --type zip
```

### Step 5 — Verify deployment

Proceed to **Section 4 — Smoke Tests** immediately after deployment.

**Record this deployment:**

| Field | Value |
|-------|-------|
| Deployed by | <!-- your name --> |
| Date/time (NZT) | <!-- YYYY-MM-DD HH:MM --> |
| Template version / Git commit | <!-- run: git rev-parse --short HEAD --> |
| Environment | <!-- dev / prod --> |
| Outcome | <!-- Success / Failed (see notes) --> |
| Notes | |

---

## 4. Post-Deployment Verification (Smoke Tests)

> Run these checks immediately after every deployment. If any check fails, proceed to Section 5 (Rollback).

### Check 1 — Resource health

```bash
# List all resources in the group and confirm state is "Succeeded"
az resource list \
  --resource-group "rg-<project>-<env>" \
  --output table

# Check App Service is running
az webapp show \
  --resource-group "rg-<project>-<env>" \
  --name "app-<project>-<env>" \
  --query "{name:name, state:state, hostName:defaultHostName}" \
  --output table
```

**Expected:** State = `Running`

### Check 2 — Application endpoint

```bash
# Test the application responds on HTTPS
curl -I "https://app-<project>-<env>.azurewebsites.net"
# OR open in browser: https://app-<project>-<env>.azurewebsites.net
```

**Expected:** HTTP status `200 OK`  
**If 503 or timeout:** App Service may still be starting; wait 2 minutes and retry.

### Check 3 — Database connectivity

```bash
# Confirm the App Service can reach the database
# Check Application Insights for connection errors
az monitor app-insights query \
  --app "appi-<project>-<env>" \
  --analytics-query "exceptions | where timestamp > ago(5m) | count"
```

**Expected:** Count = 0 (no exceptions in the last 5 minutes after deployment)

### Check 4 — Key Vault secrets accessible

```bash
# Verify the managed identity can read secrets
# (The app will fail to start if it cannot access Key Vault)
az keyvault secret list \
  --vault-name "kv-<project>-<env>" \
  --output table
```

**Expected:** Secret list returns without error.

### Check 5 — Monitoring pipeline active

```bash
# Confirm logs are flowing into Log Analytics
az monitor log-analytics query \
  --workspace "law-<project>-<env>" \
  --analytics-query "Heartbeat | where TimeGenerated > ago(5m) | count"
```

**Expected:** Count > 0 (heartbeat data received in last 5 minutes)

### Smoke Test Result

| Check | Pass ✓ | Fail ✗ | Notes |
|-------|--------|--------|-------|
| 1. Resource health | | | |
| 2. Application endpoint | | | |
| 3. Database connectivity | | | |
| 4. Key Vault accessible | | | |
| 5. Monitoring active | | | |

---

## 5. Rollback Procedure

> **When to use this:** A deployment has caused an issue and the smoke tests in Section 4 have failed. Roll back restores the previous working configuration.

> ⚠️ Inform your team before rolling back. Note the time rollback started.

### Option A — Pipeline rollback (preferred)

If your CI/CD pipeline supports deployment slots (Azure App Service deployment slots):

```bash
# Swap the production slot back to the previous version
az webapp deployment slot swap \
  --resource-group "rg-<project>-<env>" \
  --name "app-<project>-<env>" \
  --slot staging \
  --target-slot production
```

**Duration:** ~30 seconds (slot swap is near-instant)

### Option B — Re-deploy previous Git commit

```bash
# Find the previous working commit hash
git log --oneline -10

# Re-deploy the previous version by triggering pipeline on that commit
git checkout <previous-commit-hash>
git push origin HEAD:main --force-with-lease
# This triggers the pipeline to re-deploy the previous version

# OR: manual re-deployment from previous template
az deployment group create \
  --resource-group "rg-<project>-<env>" \
  --template-file infra/main.bicep \
  --parameters infra/parameters/<env>.parameters.json
```

### Option C — Infrastructure rollback

If an infrastructure change caused the failure:

```bash
# View recent deployment history
az deployment group list \
  --resource-group "rg-<project>-<env>" \
  --output table \
  --query "[].{Name:name, State:properties.provisioningState, Time:properties.timestamp}" 

# Redeploy a previous successful deployment
az deployment group create \
  --resource-group "rg-<project>-<env>" \
  --template-uri "<previous-template-url>"
  # OR re-run with the previous parameters file from git history
```

### Post-rollback

After rolling back:
1. Run all smoke tests from Section 4
2. Record the rollback in the deployment log (Section 3, Step 5 table)
3. Investigate the root cause before attempting to re-deploy

---

## 6. Restore from Backup

### 6.1 Restore Azure SQL Database (Point-in-Time)

> **When to use:** Data has been accidentally deleted or corrupted. Restores the database to a previous point in time.

> ⚠️ A point-in-time restore creates a **new** database — it does not overwrite the existing one. You must update connection strings after restore.

```bash
# Step 1: Identify the restore point
# Azure SQL retains automated backups for 7 days.
# Choose a restore time (format: YYYY-MM-DDTHH:MM:SSZ in UTC)
RESTORE_TIME="2024-10-15T08:00:00Z"   # Replace with your target time
RESTORE_DB_NAME="sqldb-<project>-restored"

# Step 2: Restore to a new database
az sql db restore \
  --resource-group "rg-<project>-<env>" \
  --server "sql-<project>-<env>" \
  --name "sqldb-<project>-<env>" \
  --dest-name "$RESTORE_DB_NAME" \
  --time "$RESTORE_TIME"

# Duration: 5–30 minutes depending on database size
```

```bash
# Step 3: Verify the restored database
az sql db show \
  --resource-group "rg-<project>-<env>" \
  --server "sql-<project>-<env>" \
  --name "$RESTORE_DB_NAME" \
  --query "{name:name, status:status}" \
  --output table
```

```bash
# Step 4: Update the connection string in Key Vault to point to the restored DB
# (Update the server/database name in the secret value)
az keyvault secret set \
  --vault-name "kv-<project>-<env>" \
  --name "SqlConnectionString" \
  --value "Server=tcp:sql-<project>-<env>.database.windows.net;Database=<RESTORE_DB_NAME>;..."
```

```bash
# Step 5: Restart the app to pick up the new connection string
az webapp restart \
  --resource-group "rg-<project>-<env>" \
  --name "app-<project>-<env>"
```

```bash
# Step 6: Run smoke tests (Section 4) to confirm application working
```

### 6.2 Restore Azure Blob Storage

```bash
# List available blob versions
az storage blob list \
  --account-name "st<project><env>" \
  --container-name "<container-name>" \
  --include "v" \
  --output table

# Restore a specific blob version
az storage blob copy start \
  --account-name "st<project><env>" \
  --destination-container "<container-name>" \
  --destination-blob "<blob-name>" \
  --source-uri "<blob-version-url>"
```

---

## 7. On-Call Health Checks

> **When to use:** Run these checks whenever you suspect an issue, after an alert fires, or as part of a regular review before a presentation or milestone submission.

### Quick Health Check (5 minutes)

Run in order. If any step fails, investigate immediately.

```bash
# 1. App Service status
az webapp show \
  --resource-group "rg-<project>-<env>" \
  --name "app-<project>-<env>" \
  --query "state" --output tsv
# Expected: Running

# 2. Recent application errors (last 30 minutes)
az monitor app-insights query \
  --app "appi-<project>-<env>" \
  --analytics-query "exceptions | where timestamp > ago(30m) | summarize count() by type | order by count_ desc"
# Expected: Empty result (no exceptions), or known/expected exceptions only

# 3. SQL Database status
az sql db show \
  --resource-group "rg-<project>-<env>" \
  --server "sql-<project>-<env>" \
  --name "sqldb-<project>-<env>" \
  --query "status" --output tsv
# Expected: Online

# 4. Active alerts
az monitor alert list \
  --resource-group "rg-<project>-<env>" \
  --output table
# Expected: No active alerts in Fired state

# 5. Recent deployment status
az deployment group list \
  --resource-group "rg-<project>-<env>" \
  --output table \
  --query "[0].{Name:name, State:properties.provisioningState, Time:properties.timestamp}"
# Expected: Most recent deployment is Succeeded
```

### Monitoring Dashboard

Access your monitoring dashboard at:  
`https://portal.azure.com/#@<your-tenant>/dashboard/arm/...`  
<!-- Paste your Azure Portal dashboard URL here after creating a shared dashboard -->

**Key metrics to review on the dashboard:**

| Metric | Healthy range | Alert threshold |
|--------|--------------|----------------|
| App CPU % | < 60% sustained | > 80% for 5 min |
| App memory % | < 70% | > 85% |
| HTTP 5xx error rate | < 1% | > 5% |
| SQL DTU/vCore % | < 70% | > 85% |
| Availability (uptime check) | 100% | < 99% |

---

## 8. Common Issues and Resolutions

| Symptom | Likely cause | Resolution |
|---------|-------------|-----------|
| App returns HTTP 503 | App Service stopped or scaling event | `az webapp start --name <app> --resource-group <rg>` then wait 2 min |
| App cannot connect to database | NSG rule blocking traffic / private endpoint misconfigured | Check NSG rules on data subnet; verify private endpoint DNS resolution |
| "Key Vault access denied" error in app logs | Managed identity missing role assignment | `az role assignment create --role "Key Vault Secrets User" --assignee <managed-identity-id> --scope <keyvault-id>` |
| Bicep deployment fails with "already exists" | Resource name conflict | Check if resource exists: `az resource list --name <resource-name>`; use `--mode Incremental` |
| Pipeline fails at "what-if" step | Service principal lacks permissions | Assign Contributor role to service principal on resource group |
| Database restore takes > 30 min | Large database size, or Azure capacity | Wait; check restore status: `az sql db show --name <restored-db>` |
| Storage account unreachable | `secureTransferRequired` enforced, using HTTP | Use HTTPS endpoints only; confirm `allowBlobPublicAccess=false` |

---

*Runbook version history: update the version number and date at the top of this file each time you make changes.*
