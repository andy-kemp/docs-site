# Quick Start Guide

Get the MFA Onboarding system deployed in under 30 minutes.

## Prerequisites

Before you begin, ensure you have:

- **Windows 10/11** or **Windows Server 2019+**
- **PowerShell 7.4+** — open `pwsh` to check (not Windows PowerShell)
- **Azure CLI** installed (`winget install -e --id Microsoft.AzureCLI`)
- **Azure Subscription** with Owner or Contributor access
- **Microsoft 365** with Global Admin or equivalent roles

!!! warning "PowerShell 7 Required"
    All scripts require PowerShell 7+ (`pwsh.exe`). Windows PowerShell 5.1 is **not supported**.
    Install it with: `winget install --id Microsoft.Powershell --source winget`

## Step 1: Download & Launch Setup

Open **PowerShell 7** (`pwsh`) and run the one-liner bootstrap command:

```powershell
irm https://raw.githubusercontent.com/andy-kemp/MFA-Onboard-Tool/main/v2/Get-MFAOnboarder.ps1 -OutFile Get-MFAOnboarder.ps1; .\Get-MFAOnboarder.ps1
```

This does three things:

1. Downloads `Get-MFAOnboarder.ps1` from GitHub
2. Downloads and extracts the full repository to your chosen install location (default: `C:\Scripts`)
3. Launches `Setup.ps1` which detects this is a fresh install and offers guided deployment

!!! tip "Alternative: Manual Download"
    If you prefer to download manually:
    ```powershell
    Invoke-WebRequest -Uri "https://github.com/andy-kemp/MFA-Onboard-Tool/archive/refs/heads/main.zip" -OutFile mfa-tool.zip
    Expand-Archive mfa-tool.zip -DestinationPath . -Force
    cd MFA-Onboard-Tool-main\v2
    .\Setup.ps1
    ```

## Step 2: Setup Menu

For a **new install**, Setup.ps1 shows:

```
========================================
  MFA Onboarding Tool - Setup
========================================

  No existing deployment found in this folder.

  What would you like to do?

    [1] New deployment   - Full guided setup (recommended)
    [0] Exit
```

Select **1** to launch the full automated deployment.

For an **existing install**, Setup.ps1 detects your config and shows:

```
  Existing deployment detected
    Tenant:       yourtenant.onmicrosoft.com
    Function App: func-mfa-yourorg-001

    [1] Update existing deployment
    [2] Pull latest scripts + update
    [3] Upgrade to v2
    [4] Fresh install (overwrite)
    [5] Resume previous install
    [6] Quick fix (pull + permissions)
    [0] Exit
```

See [Resume & Update](update.md) for details on each option.

## Step 3: Configure

The deployment wizard will prompt you for key settings, or you can pre-fill `mfa-config.ini`:

```ini
[Tenant]
TenantId=yourtenant.onmicrosoft.com
SubscriptionId=your-azure-subscription-id

[SharePoint]
SiteUrl=https://yourtenant.sharepoint.com/sites/MFAOps
SiteOwner=admin@yourtenant.com

[Security]
MFAGroupName=MFA Enabled Users

[Azure]
ResourceGroup=rg-mfa-onboarding
Region=uksouth

[Email]
MailboxName=MFA Registration
NoReplyMailbox=MFA-Registration@yourtenant.com
MailboxDelegate=admin@yourtenant.com
```

!!! tip
    See [Configuration](configuration.md) for all available settings.

## Step 4: Deployment

The full automated deployment (`Run-Complete-Deployment-Master.ps1`) runs all 8 scripts in order with retry on failure:

| Step | Script | What It Does |
|------|--------|--------------|
| 1 | `01-Install-Prerequisites.ps1` | Install PowerShell modules |
| 2 | `02-Provision-SharePoint.ps1` | Create SharePoint site & list |
| 3 | `03-Create-Shared-Mailbox.ps1` | Create shared mailbox |
| 4 | `04-Create-Azure-Resources.ps1` | Create Azure resources |
| 5 | `05-Configure-Function-App.ps1` | Configure Function App |
| 6 | `06-Deploy-Logic-App.ps1` | Deploy Logic App |
| 7 | `07-Deploy-Upload-Portal1.ps1` | Deploy upload portal |
| 8 | `08-Deploy-Email-Reports.ps1` | Deploy email reports (optional) |
| 9 | `Fix-Function-Auth.ps1` | Fix function authentication |
| 10 | `Fix-Graph-Permissions.ps1` | Fix Graph permissions |
| 11 | `Check-LogicApp-Permissions.ps1` | Fix Logic App permissions |
| 12 | `Generate-Deployment-Report.ps1` | Generate final report |

!!! note
    If interrupted, you can resume from where you left off — see [Resume & Update](update.md).

## Step 5: Post-Deployment

After deployment completes:

1. **Authorise API connections** in Azure Portal:
    - Go to Resource Group → API Connections
    - Authorise `office365` connection (sign in with an account that can send mail)
    - Authorise `office365-reports` connection (if using email reports)

2. **Test the system**:
    - Open the Upload Portal URL (shown at end of deployment)
    - Upload a test CSV or enter a test user manually
    - Check the SharePoint list for the new entry
    - Verify the invitation email arrives

## Step 6: Upload Users

Create a CSV file with your users:

```csv
UserPrincipalName,DisplayName
user1@yourtenant.com,John Smith
user2@yourtenant.com,Jane Doe
```

Upload via the web portal or use the upload function directly.

---

## What Happens Next

Once users are uploaded:

1. Logic App runs on schedule and sends invitation emails
2. Users click the tracking link in the email
3. System records the click and adds user to the MFA security group
4. On the next Logic App run, MFA registration status is checked
5. Once MFA is registered, status is set to **Active** with a timestamp

See [Tracking & Status](tracking.md) for details on each status.
