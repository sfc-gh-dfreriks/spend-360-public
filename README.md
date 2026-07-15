# SAP BDC Spend 360 — Public (anonymous) build

A static, no-login public build of the SAP BDC Spend 360 dashboard, deployed to
GitHub Pages. Dashboards render from a **point-in-time data snapshot** (no Snowflake
connection, no credentials in the browser). The optional "Ask the Agent" page calls a
separate serverless Cortex agent when configured.

## How it works

- **Dashboards:** built with `VITE_STATIC=1`. The client reads pre-baked JSON in
  `public/data/*.json` instead of a live `/api`.
- **Filter combinations:** Spend 360 filters on **categories (15)** and
  **companies (3)**. A full power set is infeasible, so the snapshot bakes companies as
  a full power set × categories {all + each single category} (128 combos). Company
  filtering is always exact; single-category and all-category views are exact; selecting
  **multiple** categories falls back to "all categories" (see `filterKey()` in
  `src/lib/api.ts`).
- **Data refresh:** re-run the exporter against the live dashboard server, commit the
  updated `public/data/*.json`, and push — Actions redeploys.
  ```bash
  # from the spend_360_react monorepo, with the server running:
  EXPORT_BASE=http://localhost:3007 node scripts/export-static.mjs
  ```
- **Live agent (optional):** set the repo variable `AGENT_URL` to a deployed Cortex
  agent Worker URL. If unset, the Analyst page shows a "not available" notice but the
  dashboards work fully.

## Local build

```bash
npm ci
VITE_STATIC=1 npx vite build      # outputs dist/
python3 -m http.server -d dist    # preview
```

## Deploy

Push to `main` → `.github/workflows/deploy.yml` builds with
`BASE_PATH=/spend-360-public/` and publishes `dist/` to GitHub Pages.

No secrets are stored in this repo. It contains only synthetic SAP demo data.
