# FAQ

Common questions about the Tenant Audit.

---

## General

### What is the Tenant Audit?

A read-only security and compliance assessment tool for Microsoft 365 tenants. It analyses your configuration across identity, data protection, devices, apps, and governance, then produces scored reports with prioritised recommendations.

### Does the audit change anything in my tenant?

**No.** The audit uses read-only application permissions only. It cannot modify users, policies, settings, or data. It reads configuration data, analyses it, and generates reports.

### How long does an audit take?

Typically **2–5 minutes** depending on tenant size. You'll see real-time progress as each category is assessed.

### How often should I run an audit?

**Monthly** is recommended. This gives you trend data showing whether your security posture is improving, and helps catch configuration drift early.

---

## Security & Privacy

### What data do you access?

Configuration data only — policies, settings, user metadata (names, sign-in dates, MFA registration status, role assignments). **We do not access email content, file content, or chat messages.**

### Where is my data stored?

Audit results are stored for **30 days** and then automatically purged. Data is not shared with third parties.

### What permissions does the app need?

The app requires approximately 25 Microsoft Graph application permissions — all read-only. See the [full permission list](getting-started.md#step-3-grant-permissions) for details.

### Can I revoke access after the audit?

Yes. You can remove the enterprise application from your Azure AD tenant at any time via **Azure Portal → Enterprise Applications**. This immediately revokes all access.

### Is the audit safe for production tenants?

Yes. The audit is designed for production environments. Read-only access means there is zero risk of configuration changes, data loss, or service disruption.

---

## Scoring & Results

### What are the three scores?

| Score | Measures |
|-------|----------|
| **Security Risk** (0–100) | Overall security posture based on identity, data, device, and app configuration |
| **Compliance Posture** (0–100) | Alignment with NCSC CAF, Cyber Essentials, CE+, and NIST 800-53 |
| **Migration Complexity** (0–100) | Estimated complexity of migrating/consolidating the tenant |

### Why is my compliance score low?

Common reasons:

- **Missing licensing** — Azure AD P2 features (PIM, risk-based policies) can't be assessed without the licence
- **Security Defaults only** — Tenants relying on Security Defaults rather than Conditional Access will have many compliance controls marked as "Not Assessed"
- **No device management** — Without Intune, device compliance controls can't be evaluated

### Some controls show "Not Assessed" — what does that mean?

The feature required to evaluate that control isn't available — usually because the tenant doesn't have the necessary Microsoft 365 licence. See [Licensing Requirements](licensing.md) for what's needed.

### Can I improve my score quickly?

Yes — the audit produces a prioritised action plan. **Quick wins** (high impact, low effort) are listed first. Common quick wins include:

- Removing unnecessary Global Admins
- Disabling stale accounts
- Converting CA policies from report-only to enabled
- Resolving expired app credentials

---

## Licensing

### Do I need Microsoft 365 E5 to use the audit?

**No.** The audit runs on any Microsoft 365 tenant. However, higher licence tiers unlock more checks and deeper compliance assessments. See [Licensing Requirements](licensing.md) for a full breakdown.

### What if I can't upgrade my licensing?

You have two options:

1. **Run the audit as-is** — You'll still get valuable insights into identity hygiene, mailbox security, and migration complexity
2. **Engage Andy Kemp Consulting** — We can supplement the automated audit with manual assessment to cover gaps. [Learn more](licensing.md#option-2-engage-andy-kemp-consulting)

---

## Reports

### What report formats are available?

Three formats: **Technical PDF** (detailed), **CXO Executive PDF** (business-focused), and **CXO PowerPoint** (presentation-ready). All are generated from the same audit data.

### Can I customise the reports?

The PDF reports are generated automatically. The **PowerPoint (PPTX) report** is fully editable — you can add your own branding, commentary, and additional slides.

### How long are reports available?

Reports are available for **30 days** after the audit completes. Download them promptly or re-run the audit when needed.

---

## Support

### I need help interpreting my results

[Andy Kemp Consulting](licensing.md#option-2-engage-andy-kemp-consulting) offers managed audit services including expert interpretation, remediation planning, and implementation support.

### I found a bug or have a feature request

Open an issue on the [:material-github: GitHub repository](https://github.com/andy-kemp/andykemp-audit/issues).

### How do I contact Andy Kemp Consulting?

[:material-email: andy@andykemp.com](mailto:andy@andykemp.com) or visit [:material-web: andykemp.com](https://andykemp.com).
