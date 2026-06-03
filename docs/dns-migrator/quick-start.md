# Quick Start

Get DNS Migrator running in under 5 minutes.

## Option A: Web UI (Cloudflare Pages)

The fastest way — deploy to Cloudflare Pages with zero infrastructure.

### 1. Clone the repository

```bash
git clone https://github.com/andy-kemp/DNS-Migrator.git
cd DNS-Migrator
```

### 2. Install dependencies

```bash
npm install
```

### 3. Deploy to Cloudflare Pages

```bash
npx wrangler pages deploy public
```

That's it — your DNS Migrator is live. Open the URL provided by Wrangler.

### 4. Start migrating

1. Enter your **Cloudflare API token** and click **Validate**
2. Choose a DNS source (Scan, Azure, or Manual)
3. Select the zones you want to migrate
4. Click **Migrate** and watch records stream in real-time
5. Update your registrar with the nameservers shown

---

## Option B: Run Locally

### 1. Clone and install

```bash
git clone https://github.com/andy-kemp/DNS-Migrator.git
cd DNS-Migrator
npm install
```

### 2. Start the server

=== "Express (simple)"

    ```bash
    npm start
    ```

=== "Wrangler (Cloudflare runtime)"

    ```bash
    npm run pages:dev
    ```

### 3. Open the UI

Navigate to [http://localhost:3000](http://localhost:3000) and follow the wizard.

---

## Option C: PowerShell CLI

For scripted or automated migrations from Azure DNS.

### Single zone

```powershell
.\Migrate-DNS.ps1 `
  -AzureResourceGroup "my-dns-rg" `
  -ZoneName "example.com" `
  -CloudflareApiToken "your-cf-token" `
  -CloudflareZoneId "your-zone-id" `
  -DryRun
```

Remove `-DryRun` when you're ready to create records for real.

### Multiple zones

1. Copy `config.example.json` to `config.json` and fill in your zones
2. Run:

```powershell
.\Migrate-Batch.ps1 `
  -ConfigFile ".\config.json" `
  -CloudflareApiToken "your-cf-token" `
  -DryRun
```

## Next Steps

- [Prerequisites](prerequisites.md) — Full list of requirements
- [Web UI Guide](web-ui.md) — Detailed walkthrough of each step
- [PowerShell CLI](powershell.md) — All parameters and examples
