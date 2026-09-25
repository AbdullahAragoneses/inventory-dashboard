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
- `index.html` — the shell. `PROJECT_SUMMARY.md` — this file.
- `courier/index.html` — Courier Fees section (added 2026-09-25). A page hosted in this repo,
  loaded in the same iframe as the external apps (relative `url: 'courier/'` in `APPS`).

## Courier Fees section (2026-09-25)
Three courier tabs: **Brinks** (built), **RAM** and **MDS** (placeholders until their sheets are mapped).

**Brinks** replaces `11 Business Associates\BRINKS\BRINKS CALCULATOR.xlsx`, tab `BRINKS CALCULATOR`
(Abdullah called it "CURRENT"; no tab of that exact name exists in the saved file). Every rate is in the
`DEFAULT_RATES` object with the source cell in a comment. Logic, verified against the sheet's own
numbers (140 KR @ R73,300 to JHB = R8,150.76, the sheet's TOTAL):
- Delivery = flat rate per destination (JHB / Pretoria R2,500; CPT consolidated R5,375; CPT not
  consolidated R7,500) + 15% VAT.
- Insurance = 0.048% of insured value; insured value = spot, or spot +4.5% (gold) / +8% (silver)
  when "Insure at" is set to a replacement uplift.
- Admin fee R350 (toggle; the sheet notes it is only charged if Brinks opens the package).
- Weight surcharge R39.95/kg excl VAT for each started kg over 4 kg, floored at zero (the sheet's
  own "add IF, if negative then zero" note). OFF by default because the sheet says weight charges
  don't apply to local movement.
- "VAT on insurance & admin too" toggle: OFF reproduces the sheet's Summary of Cost (VAT on delivery
  only); ON reproduces its side block F26 (VAT on everything). **Open question for Abdullah: which
  one matches the real Brinks invoice?**
Rate card is editable in the page and saved per browser (`localStorage courier.brinks.rates.v1`),
"Reset to sheet values" restores the defaults. Test harness: a Node script that extracts `calc()`
from the page and checks 11 sheet cells (lived in the session scratchpad; re-create from the cell
comments if needed).

## Known limitation — browser file pickers inside frames
Chrome will not open folder/file pickers from an app embedded in another site's frame. Stock
Take's Save/File detects this (since 2026-09-11) and opens itself in a new tab on History &
Photos instead. Any future embedded app that uses the File System Access API needs the same guard.

## Pending / ideas
- Share the link with Zubayr and Angie; retire the old Stock Take URL on the personal account.
- Optional: single shared login across Proof + Stock Take (both use the same name+PIN pattern).
- Optional: Merino cream (#F5EEDD) page background variant — Abdullah preferred white for now.
