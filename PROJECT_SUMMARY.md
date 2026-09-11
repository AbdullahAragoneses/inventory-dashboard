# Inventory Dashboard

## What it is
One page with a collapsible side menu that opens each of the inventory team's tools inside the
same window, so the team has one link instead of three: Proof Coin Dashboard, Stock Take
Dashboard, SLI Generator. Live since 2026-09-11.

**Live link (share this):** https://abdullaharagoneses.github.io/inventory-dashboard/
Repo: `AbdullahAragoneses/inventory-dashboard` (public, GitHub Pages from `master`).

## How it works (deliberately simple)
`index.html` is a single static file. The side menu and home cards are generated from the `APPS`
list near the top of the `<script>`. Choosing a tool loads its live URL into an `<iframe>`; the
apps themselves are untouched. Light/dark toggle (top-right) is pushed into the embedded app via
`postMessage({type:'invdash-theme'})`, which all three apps listen for. Remembers last tool,
menu state and theme per browser.

**To add a new app:** add one line to `APPS` (id, icon, label, url, blurb, tint). Commit + push;
Pages redeploys in about a minute.

**Local preview:** `python -m http.server 8787` in this folder, then
`http://localhost:8787/?local` — `?local` swaps the app URLs to localhost dev servers
(Proof 8789 static, SLI 8788 static, Stock Take 8000 uvicorn). `&theme=dark` forces a theme.

## Design
Second reference Abdullah supplied (light sidebar, grey line borders, few colours), recoloured
with his brand blues: **Venice Blue #16587B** primary, **Rock Blue #84B3CE** lighter companion,
Tailwind grey scale for everything else. Both modes. The three apps share the exact same tokens
(their `:root` / `[data-theme="light"]` blocks were rewritten 2026-09-11) so the whole window
reads as one product.

## Where the apps live (2026-09-11)
| Tool | Live URL | Hosting |
|---|---|---|
| Proof Coin Dashboard | https://proof-dashboard-eight.vercel.app | Vercel, team **Bullion Boeties** (Pro) |
| Stock Take Dashboard | https://stock-take-dashboard-seven.vercel.app | Vercel, team **Bullion Boeties** (Pro) |
| SLI Generator | https://abdullaharagoneses.github.io/sli-generator/ | GitHub Pages |

The old personal-account Stock Take URL (`stock-take-dashboard.vercel.app`) still resolves but
serves the OLD look and the suspended Blob store — retire it; the dashboard points at `-seven`.

## Key files
- `index.html` — the whole app. `PROJECT_SUMMARY.md` — this file.

## Pending / ideas
- Share the link with Zubayr and Angie; retire the old Stock Take URL on the personal account.
- Optional: single shared login across Proof + Stock Take (both use the same name+PIN pattern).
- Optional: Merino cream (#F5EEDD) page background variant — Abdullah preferred white for now.
