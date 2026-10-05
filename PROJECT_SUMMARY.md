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

## Design (since 2026-09-25: Solstice look)
Abdullah asked for the hub to look like Solstice (solstice.goldstore.co.za). Tokens were captured
live from the logged-in Solstice dashboard via Chrome DevTools: it is the **Minimal UI** design
system. Font **Public Sans**; text #1C252E / #637381 / #919EAB; grey scale 100 #F9FAFB → 900
#141A21; primary = dark #1C252E / #141A21 (nav active = #141A21 with white text, radius 8, or
`0 15px 15px 0` in the 88 px mini rail); success #0BDDB2 / #099E7F; warning #FF9E01; info #00A8D5;
secondary #8032FD; cards radius 16, no border, shadow `0 0 2px rgba(145,158,171,.2), 0 12px 24px
-4px rgba(145,158,171,.12)`; dark hero cards `linear-gradient(135deg,#0B0F19,#111827 50%,#0D1322)`.
Sidebar 300 px expanded with a 68 px black logo strip (Solstice logo `assets/logo.svg`), round
chevron toggle on the edge, green-ringed avatar bottom-left. Nav icons are the Solstice navbar set
(`assets/icons/ic-*.svg`, used as CSS masks so they take the text colour). Dark mode uses the
Minimal dark palette (#141A21 / #1C252E / #919EAB). Home page mirrors Solstice's hero: a LIVE SAST
TIME clock card and a LIVE SPOT BULLION (ZAR) card fed by the same free APIs as the courier page.

**The three embedded apps still wear the older grey + Venice Blue theme** (rewritten 2026-09-11);
re-theming them to Public Sans + Minimal tokens is the next step if Abdullah wants one look inside
the frames too.

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
**Multi-line shipments (same day, Abdullah's ask):** a shipment is a list of lines, each Metal ×
Size (1 oz, 1/2, 1/4, 1/10 oz, 1 kg bar, 100 g bar, custom grams) × Units × Unit price, so gold and
silver travel in one quote. Spot value, insured value (per-metal uplift: gold 4.5%, silver 8%,
platinum 4.5% ASSUMED, not in the sheet) and weight are summed across lines. "Insure at replacement
value" is now a switch instead of the old per-shipment dropdown. Inputs are stored under
`courier.brinks.inputs.v2` (v1 shape is ignored). Test: `test_calc2.mjs` in the session scratchpad,
21 checks incl. a 140 gold + 500 silver joint shipment.
**Live spot:** unit price = live metal price per oz × unit weight in oz. Gold/silver/
platinum USD per oz from `api.gold-api.com` (free, no key, CORS open) × USD→ZAR from `open.er-api.com`
(free, updates daily). Refreshes every 5 minutes; typing in the price switches to MANUAL with a
"Use live price" link back. If either endpoint dies, the page says so and the price stays editable.
**Quote card** button opens a white, screenshot-ready breakdown (facts grid + line table + big total);
`?quote` in the URL opens it on load for previews.
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

## RAM calculator (2026-09-25, rebuilt from Outlook quotes the same day)
First version copied `abdullahs storage
. RAM\RAM Insurance\RAM_Courier_Calculator.xlsx`. It was then rebuilt
from **24 real RAM quotes** on account AURU02 found in Outlook (Mandre Pretorius, Peter van der Berg, Abram Maboa,
Odette Strydom; Sep 2024 to Aug 2026). The workbook's model was wrong in several ways the quotes prove:
- **Base charge depends on chargeable weight, not route** (R1,883.11 at 7.2 kg local, regional and Durban alike).
  Weight bands in `RAM_RATES.bands`: 2.4 kg R1,048.01 and 7.2 kg R1,883.11 (quoted 2026); 9.6 kg and 20 kg are
  2025 quotes plus RAM's 6% April 2026 increase. Between points = straight-line ESTIMATE, flagged on the page.
- Fuel is a monthly % of base (51.73% on 31 Aug 2026; history on the page). Waybill R29.20, AV R9,249.11.
- **Face to face is flat R129.13**, charged even without AV. **Part 108 R70 always** ("known shipper", Mandre 31 Aug).
- **AV rule:** needed above R150k, **waived under R1m when collected and delivered in the same city, JHB or CPT**
  (Odette 13 Nov 2025), forced above R5m by our specie policy. East London: armed escort, same price. Manual override.
- Liability 0.4% of value, minimum R50, **needs RAM sign-off** (no standing cover; directors refused R10m in May 2026).
- RAM rounds each line to the cent; so does the calculator.
Verified: `test_ram.mjs` (scratchpad) reproduces 8 RAM quotes to the cent (Montana Gardens, Hermanus x3, Umhlanga,
Ballito, Gillitts x2) plus the rules. Default inputs = latest quote, 31 Aug 2026 Bedfordview to Gillitts, R17,097.73.
Hubs are chosen From/To (Durban = DBN). Client mark-up and own-vehicle cover (0.035%, R5m cap) cards kept.
**Still open:** RAM rate card PDF expired 2026-08-31; RAM's portal quotes price base lower than Mandre's emailed ones
(he says the email is correct); more quoted weights would replace the estimated bands.

## Logo (2026-09-25)
`assets/logo.svg`: circle + two vertical ticks + letter-spaced INVENTORY, from Abdullah's image.
Used as a CSS mask (white on the dark strip), favicon, and in the quote card headers.

## MDS Collivery calculator (2026-09-25)
Built from the MDS portal (collivery.net, account SABIS; Abdullah logged in, read only):
- **Rate sheet PDF** (Prices > Rate Sheet, printed 2026-09-25; copy in Downloads as `MDS Rate Sheet SABIS 2026-09-25.pdf`):
  per-kg matrix for 32 hubs x 4 services (Same Day SDX, Next Day ONX, Road Freight Express ECO, Road Freight FRT),
  minimums (Local / Major / Main / Inter main), included kg, volumetric divisor, surcharges, location types.
- **Waybill history CSV** (Administration > Waybill History > Export CSV, Apr-Sep 2026, 758 waybills, itemised).
  Kept only in Downloads/scratchpad: it has client addresses, never put it in this public repo.
Formula (matches 742/758 waybills to the cent, all within R0.20): base = sheet minimum / 0.89; extra kg = started kg
above included (2 kg major/main, 15 kg local ONX) x (sheet per-kg / 0.89, + R11.21 if regional/outlying); regional
R136.80 or outlying R152.30; 11% discount off base + extra + regional; + R10.20 doc fee + town surcharge + time
surcharge + R0.50 delivery PIN; fuel 26.8% on all of that; + location-type fee (no fuel); rounded to nearest 20c.
Town -> area type lookup (~150 towns) and remote-town surcharges (Beaufort West R561.80, Chintsa East R150, Kokstad
R120, Prieska R100, Phalaborwa R60, Lime Acres R25) come from the history. MDS vs RWS (sister company) delivery areas
from Kathy Bergoff's 16 Sep 2026 email; PIN delivery only in MDS branch areas. Test: `test_mds.mjs` (scratchpad),
10 real waybills. **Watch:** fuel changes monthly (24.6% Apr to 29.4% Jun); MDS re-weighs parcels (2.2 kg volumetric
turned R188 into R270.20).

**MDS suburbs (same day, Abdullah's ask):** `courier/mds-suburbs.json` = the MDS portal's Suburbs export
(Administration > Suburbs > Export, 25 Sep 2026): 9,828 suburbs in 2,126 towns with category (Major/Main/Regional/
Outlying) and MDS's High Risk flag. Each town is assigned to its nearest of the 32 rate-sheet hubs by coordinates, so
the Delivery suburb list only offers suburbs served from the chosen Deliver-to hub. Collection area, delivery area,
rate type (Local/Major/Main/Regional/Outlying), high-risk location fee (R130) and the remote town surcharge now all
fill in by themselves. Town surcharges are not published by MDS, so only the six towns seen on invoices are known
(Beaufort West R561.80, Chintsa East R150, Kokstad R120, Prieska R100, Phalaborwa R60, Lime Acres R25); other
Regional/Outlying towns show a "check" note.

## BRINKS rebuilt from real invoices (2026-09-26)
Shahiema's recon is not one sheet: a workbook per month (`Brinks_Reconciliation_<Month>_2026.xlsx` etc.) filed with the
invoices in `23 SABIS Accounting. Invoices and PoP's6\<NN MONTH>\<nn>. BRINKS\`. The recon has no SLI numbers;
the **Brinks invoice PDFs** do ("Invoice #: GLD 10159", with a space). A read-only extraction of every 2026 invoice
(349 documents, 415 charge lines, 130 invoices / 121 SLIs with a GLD number) is saved at
`Projects\Reference Documents\Brinks invoices 2026\` (CSV + notes + the three test scripts). Not in this public repo.
What the invoices prove (and the calculator now uses):
- Route-based flat charge: in-house Bedfordview move R350 admin only, no liability (101 of the SLIs); Bedfordview to
  Sandton / Pretoria / Centurion R2,500; Rand Refinery R1,650 (R1,833 in Jan); Brinks Kempton Park R1,833; Cape Town to
  Johannesburg door to door R6,312; airfreight to Cape Town R5,375 consolidated / R7,500 not.
- Liability 0.048% of the declared (customs) value, minimum USD 50 at the invoice rate, truncated to the cent.
- Weight charges R45 per kg of GROSS weight on intercity freight (not R39.95 over 4 kg as in the sheet).
- 15% VAT on every line, liability included (the sheet's "VAT on delivery only" total was wrong).
Test: `test_brinks.mjs` re-prices 12 real SLI invoices to the cent. The old sheet-based BRINKS test is retired.
Worth raising with Brinks: Rand Refinery liability billed three ways (10041 0.048%, 10071 minimum only, 10059 none);
GLD 10165 R1,475.00 vs 0.048% = R1,475.13; two liability lines at 0.045%; two "ANNUAL LIABILITY 0.02%" at 0.068%;
Jan statement invoices 180755-180874 and 181355/181368 have no PDF in SharePoint.

## PUBLIC REPO WARNING
`inventory-dashboard` is a PUBLIC GitHub repo served on GitHub Pages. The courier page now embeds negotiated account
pricing (MDS rate sheet, RAM quote figures, BRINKS rates). Pushing publishes them. Decide before pushing: make the
repo private (Pages then needs a paid plan) or host the courier page somewhere access-controlled.

## 2026-09-25 teal trial (uncommitted)
- Black accents replaced with Venice Blue teal shades in the hub, Courier Fees and the three embedded apps (local copies); header bars use the teal sheen gradient.
- MDS: "Collection point" picker (SA Bullion Bedfordview / BRINKS Bedfordview / SA Bullion Woodstock / Other) fills hub + suburb; default parcel 0.5 kg, 30×20×10 cm.
- Courier input column widened to 580px max.
- Pending: Abdullah's approval, then commit + deploy per app. Ideas offered: collection point name on the MDS quote card; same picker on RAM.

## Serial Numbers section (2026-09-26)
Sidebar entry `serials` opens the Serial Number Trackers app (1kg / 500g SABIS bars + OP coin; repo
`AbdullahAragoneses/serial-number-trackers`, private). It runs **only on Abdullah's PC for now**
(http://127.0.0.1:8790, started by Task Scheduler), so the entry carries `localOnly: true` and is hidden
on the public site: it shows on 127.0.0.1/localhost or with `?local`. When the app is hosted (Vercel +
Turso), set its real URL in `APPS` and drop `localOnly`.

## Product Uploads (2026-10-05)
Sidebar entry `products` opens **https://product-uploads.vercel.app** — a separate PRIVATE app (repo
`AbdullahAragoneses/product-uploads`, Vercel team bullion-boeties, private Blob store in Cape Town) with its own
name + password login. Nothing about the products lives in this public repo; see that repo's PROJECT_SUMMARY.md.
`serve.py` here (Task Scheduler "Inventory Dashboard - Server") still serves the hub preview on 127.0.0.1:8787; it is
local only and not committed. The earlier local version of the page is retired.
