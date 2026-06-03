# DNS Migrator

Migrate DNS zones from any provider to Cloudflare — cleanly, correctly, and without record mangling.

## What It Does

DNS Migrator solves the critical problem of DNS record corruption during zone transfers. When importing DNS records, relative names like `home` often become `home.example.com` and resolve incorrectly as `home.example.com.example.com`. DNS Migrator strips zone suffixes, removes trailing dots, and pushes clean, correctly-formatted records to Cloudflare via API.

## How It Works

```
Choose DNS Source (Scan / Azure / Manual)
        ↓
Discover or import records
        ↓
Select zones to migrate
        ↓
DNS Migrator creates zones in Cloudflare
        ↓
Records are pushed via Cloudflare API
        ↓
Nameservers displayed for registrar update
```

## Three Ways to Run

| Mode | Best For | How |
|------|----------|-----|
| **Web UI** | Interactive migrations with live feedback | Cloudflare Pages or local Node.js |
| **PowerShell (Single)** | Scripted single-zone migration | `Migrate-DNS.ps1` |
| **PowerShell (Batch)** | Bulk migration of many zones | `Migrate-Batch.ps1` with JSON config |

## Three DNS Sources

| Source | Description |
|--------|-------------|
| **DNS Scan** | Probes live DNS via DNS-over-HTTPS — works with any provider |
| **Azure DNS** | Full zone export via Azure Management API — captures all records |
| **Manual** | Enter domain names only — creates empty zones in Cloudflare |

## Key Features

- :material-cloud-sync: **Any provider to Cloudflare** — scan live DNS from GoDaddy, Route53, Azure DNS, Namecheap, etc.
- :material-shield-check: **Clean record transformation** — strips FQDNs, removes trailing dots, validates format
- :material-content-duplicate: **Duplicate detection** — compares existing Cloudflare records before insertion
- :material-test-tube: **Dry-run mode** — preview what would be created without making changes
- :material-format-list-checks: **Batch operations** — process multiple zones from a JSON config
- :material-progress-clock: **Real-time streaming** — live NDJSON feedback for every record created
- :material-dns: **Broad record support** — A, AAAA, CNAME, MX, TXT, SRV, CAA, NS, PTR

## Quick Links

- [Quick Start](quick-start.md) — Get up and running fast
- [Architecture](architecture.md) — How it all fits together
- [Prerequisites](prerequisites.md) — What you need before starting
- [Deployment](deployment.md) — Deploy to Cloudflare Pages or run locally
- [Web UI Guide](web-ui.md) — Step-by-step walkthrough of the web interface
- [PowerShell CLI](powershell.md) — Command-line migration scripts
- [API Reference](api-reference.md) — REST endpoint documentation
- [Troubleshooting](troubleshooting.md) — Common issues and fixes

## Source Code

[:material-github: GitHub Repository](https://github.com/andy-kemp/DNS-Migrator){ .md-button }
