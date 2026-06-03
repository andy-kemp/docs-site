# Deployment

How to deploy DNS Migrator for each supported mode.

## Cloudflare Pages (Recommended)

The simplest deployment — serverless, free tier, auto-scaling.

### 1. Clone and install

```bash
git clone https://github.com/andy-kemp/DNS-Migrator.git
cd DNS-Migrator
npm install
```

### 2. Deploy

```bash
npx wrangler pages deploy public
```

Wrangler will:

- Upload `public/` as static assets (the web UI)
- Deploy `functions/` as serverless API endpoints
- Return a live URL (e.g. `https://dns-migrator.pages.dev`)

### 3. (Optional) Connect to GitHub

For automatic deployments on push:

1. Go to [Cloudflare Dashboard → Pages](https://dash.cloudflare.com/?to=/:account/pages)
2. Click **Create a project** → **Connect to Git**
3. Select the `DNS-Migrator` repository
4. Set build output directory to `public`
5. Deploy

Every push to `main` will trigger a new deployment automatically.

---

## Local Node.js (Development)

Run DNS Migrator on your local machine with Express.

### 1. Clone and install

```bash
git clone https://github.com/andy-kemp/DNS-Migrator.git
cd DNS-Migrator
npm install
```

### 2. Start the server

```bash
npm start
```

The server starts on `http://localhost:3000` (or the port set via `PORT` environment variable).

### 3. Access the UI

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## Local Wrangler Dev

Run the Cloudflare Workers runtime locally — useful for testing before deploying to Pages.

```bash
npm install
npm run pages:dev
```

This starts a local dev server at `http://localhost:3000` using the Wrangler runtime, which closely matches the Cloudflare Pages production environment.

---

## PowerShell (No Deployment Needed)

The PowerShell scripts run directly from the cloned repository — no server or deployment required.

```bash
git clone https://github.com/andy-kemp/DNS-Migrator.git
cd DNS-Migrator
```

Then run `Migrate-DNS.ps1` or `Migrate-Batch.ps1` as described in the [PowerShell CLI](powershell.md) guide.

---

## Deployment Summary

| Mode | Deploy Command | URL |
|------|---------------|-----|
| Cloudflare Pages | `npx wrangler pages deploy public` | `*.pages.dev` |
| Local Express | `npm start` | `http://localhost:3000` |
| Local Wrangler | `npm run pages:dev` | `http://localhost:3000` |
| PowerShell | N/A | N/A (CLI only) |
