# Resume & Update

## Updating an Existing Deployment

There are three ways to update: Setup.ps1 (recommended), Update-Deployment.ps1 (advanced), or a one-liner download command.

### Option 1: Setup.ps1 (Recommended)

From the `v2` folder, run:

```powershell
.\Setup.ps1
```

Setup.ps1 detects your existing deployment and shows:

```
  Existing deployment detected
    Tenant:       yourtenant.onmicrosoft.com
    Function App: func-mfa-yourorg-001

    [1] Update existing deployment    - Change branding, redeploy code, fix permissions
    [2] Pull latest scripts + update  - Download newest code from GitHub, then update
    [3] Upgrade to v2                 - Full upgrade: schema, functions, Logic App, permissions
    [4] Fresh install (overwrite)     - Full deployment from scratch
    [5] Resume previous install       - Continue where the last install left off
    [6] Quick fix (pull + permissions) - Download latest code and fix all permissions
    [0] Exit
```

| Option | When to Use |
|--------|-------------|
| **[1] Update existing deployment** | You already have the latest scripts and want to redeploy a component |
| **[2] Pull latest scripts + update** | You want to download the latest code from GitHub and then choose what to update |
| **[3] Upgrade to v2** | You're on v1 and want to migrate to v2 (schema + tokens + functions + Logic App + permissions) |
| **[4] Fresh install** | You want to completely redo the deployment from scratch (backs up config first) |
| **[5] Resume previous install** | A previous deployment was interrupted and you want to continue from where it stopped |
| **[6] Quick fix** | One-click: downloads latest code, then fixes Function Auth + Graph Permissions + Logic App Permissions |

!!! tip "Most common choice"
    Use **[2] Pull latest scripts + update** to get the newest code and then pick which components to redeploy. This is the safest way to apply updates.

### Option 2: Update-Deployment.ps1 (CLI Switches)

For automation or when you know exactly what to update:

```powershell
# Interactive menu
.\Update-Deployment.ps1

# Update everything
.\Update-Deployment.ps1 -UpdateAll

# Update specific components
.\Update-Deployment.ps1 -FunctionCode     # Redeploy function code only
.\Update-Deployment.ps1 -LogicApp         # Redeploy Logic App template
.\Update-Deployment.ps1 -Branding         # Update branding in emails
.\Update-Deployment.ps1 -Permissions      # Reapply Graph permissions
.\Update-Deployment.ps1 -SharePointSchema # Update SharePoint columns
.\Update-Deployment.ps1 -BackfillTokens   # Generate tracking tokens for existing users

# Quick fix: Function Auth + Graph Permissions + Logic App Permissions
.\Update-Deployment.ps1 -QuickFix

# Full v2 upgrade (schema + backfill + functions + Logic App + permissions)
.\Update-Deployment.ps1 -Upgrade
```

### Option 3: One-Liner Download + Update

To update the scripts on a deployment machine without interactive prompts:

```powershell
cd C:\path\to\MFA-Onboard-Tool-main
```

Then run this one-liner (preserves your `mfa-config.ini`):

```powershell
iwr https://github.com/andy-kemp/MFA-Onboard-Tool/archive/refs/heads/main.zip -OutFile $env:TEMP\mfa-update.zip; Expand-Archive $env:TEMP\mfa-update.zip $env:TEMP\mfa-update -Force; if (Test-Path .\v2\mfa-config.ini) { Copy-Item .\v2\mfa-config.ini $env:TEMP\mfa-config.ini.bak }; Remove-Item .\v2 -Recurse -Force -ErrorAction SilentlyContinue; Copy-Item $env:TEMP\mfa-update\MFA-Onboard-Tool-main\v2 .\v2 -Recurse -Force; if (Test-Path $env:TEMP\mfa-config.ini.bak) { Copy-Item $env:TEMP\mfa-config.ini.bak .\v2\mfa-config.ini -Force; Remove-Item $env:TEMP\mfa-config.ini.bak }; Remove-Item $env:TEMP\mfa-update,$env:TEMP\mfa-update.zip -Recurse -Force
```

Then apply the update:

```powershell
cd v2
.\Update-Deployment.ps1 -LogicApp   # or whichever component needs updating
```

---

## Resuming a Failed Deployment

### Via Setup.ps1

```powershell
.\Setup.ps1
# Choose option [5] Resume previous install
```

Setup.ps1 detects the last completed step and shows it in the menu.

### Via CLI

```powershell
# Automatic resume from last completed step
.\Run-Complete-Deployment-Master.ps1 -Resume

# Resume from a specific step
.\Run-Complete-Deployment-Master.ps1 -StartFromStep 7
```

Deployment state is saved to `logs\deployment-state.json` after each step.

### Step Reference

| Step | Script | Description |
|------|--------|-------------|
| 1 | `01-Install-Prerequisites.ps1` | Install PowerShell modules |
| 2 | `02-Provision-SharePoint.ps1` | Create SharePoint site & list |
| 3 | `03-Create-Shared-Mailbox.ps1` | Create shared mailbox |
| 4 | `04-Create-Azure-Resources.ps1` | Create Azure resources |
| 5 | `05-Configure-Function-App.ps1` | Configure Function App |
| 6 | `06-Deploy-Logic-App.ps1` | Deploy Logic App |
| 7 | `07-Deploy-Upload-Portal1.ps1` | Deploy upload portal |
| 8 | `08-Deploy-Email-Reports.ps1` | Deploy email reports |
| 9 | `Fix-Function-Auth.ps1` | Fix function authentication |
| 10 | `Fix-Graph-Permissions.ps1` | Fix Graph permissions |
| 11 | `Check-LogicApp-Permissions.ps1` | Fix Logic App permissions |
| 12 | `Generate-Deployment-Report.ps1` | Generate final report |

---

## Upgrading from v1 to v2

If you have an existing v1 installation:

### Via Setup.ps1 (Recommended)

```powershell
.\Setup.ps1
# Choose option [3] Upgrade to v2
```

This automatically pulls the latest scripts from GitHub, then runs the full upgrade.

### Via CLI

```powershell
.\Update-Deployment.ps1 -Upgrade
```

### What the Upgrade Does

The upgrade performs 5 steps in order:

1. **SharePoint Schema** — Adds new columns (`TrackingToken`, `LastChecked`, `MFARegistrationDate`, etc.)
2. **Backfill Tokens** — Generates GUID tracking tokens for all existing users
3. **Function Code** — Deploys updated function code with token-based tracking
4. **Logic App** — Redeploys Logic App with token-based URLs and MFA date tracking
5. **Permissions** — Reapplies all Graph API permissions for Managed Identities

!!! note
    The upgrade is safe to run multiple times — all operations are idempotent.
