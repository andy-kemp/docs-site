# MFA Onboarding System

Automated Multi-Factor Authentication enrollment for Microsoft 365 environments.

**Version:** 2.0

## What It Does

The MFA Onboarding system provides an end-to-end automated workflow for rolling out MFA across your organisation:

1. **Upload users** via a web portal (CSV drag-and-drop, manual entry, or batch upload)
2. **Send branded emails** with personalised enrollment instructions, tracking links, and App Store links
3. **Track engagement** — email opens (invisible pixel), link clicks, MFA registration dates
4. **Automatically manage** security group membership for Conditional Access
5. **Send reminders** automatically after 7 days, escalating to the user's manager after 2+ reminders
6. **Self-service resend** — users can request a new invitation without contacting IT
7. **Report to admins** via daily/weekly email reports and a real-time portal dashboard with CSV export

## User Journey

```
Administrator uploads CSV (or enters manually)
        ↓
Logic App sends branded email → User receives email
        ↓                                    ↓
SharePoint status: "Sent"          Email open tracked (pixel)
                                             ↓
                                   User clicks enrolment link
                                             ↓
                                   Function App records click
                                   + adds to security group
                                   + shows branded confirmation page
                                             ↓
                                   SharePoint status: "AddedToGroup"
                                             ↓
                                   Logic App checks MFA registration
                                             ↓
                                   SharePoint status: "Active" ✓
                                   
                                   If not enrolled after reminders:
                                   → Manager escalation email sent
```

## Components

| Component | Purpose |
|-----------|---------|
| **Upload Portal** | Web SPA with drag-and-drop CSV upload, manual entry, reports dashboard, and CSV export |
| **Azure Function App** | PowerShell 7.4 with 4 HTTP endpoints: `upload-users`, `enrol`, `track-open`, `resend` |
| **Logic App (Invitations)** | Sends emails, checks MFA status, manages reminders, escalates to managers |
| **Logic App (Reports)** | Sends daily/weekly executive summary reports to admins |
| **Application Insights** | Full telemetry, request logging, exception tracking, live metrics |
| **SharePoint List** | Central tracking database with 24 columns per user |
| **Security Group** | Managed automatically for Conditional Access policies |
| **Shared Mailbox** | Sends branded enrollment emails from a custom address |

## Key Features

| Feature | Description |
|---------|-------------|
| **Token-based tracking** | GUID tokens in URLs instead of email addresses — no PII exposure |
| **Email open tracking** | Invisible pixel records when emails are opened |
| **Manager escalation** | Automatic escalation after 2+ unanswered reminders |
| **Self-service resend** | Users can request a new link without IT involvement |
| **Branded HTML pages** | Styled response pages for all tracking link outcomes |
| **Retry policies** | Exponential backoff on all Logic App API actions |
| **Application Insights** | Full telemetry and monitoring for Function App |
| **Infrastructure as Code** | Bicep templates for all Azure resources |
| **Resume & update** | Deployment can be resumed, updated, or upgraded without redeploying everything |

## Quick Links

- [Quick Start Guide](quick-start.md) — Get up and running fast
- [Architecture](architecture.md) — How it all fits together
- [Deployment](deployment.md) — Step-by-step deployment guide
- [Upload Portal](upload-portal.md) — Web interface guide
- [Tracking & Status](tracking.md) — Status lifecycle and SharePoint schema
- [Troubleshooting](troubleshooting.md) — Common issues and fixes
- [What's New](whats-new.md) — Latest features and changes

## Source Code

[:material-github: GitHub Repository](https://github.com/andy-kemp/MFA-Onboard-Tool){ .md-button }
