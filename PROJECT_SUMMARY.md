# Inventory Hub

## What it is
One page with a collapsible side menu (modelled on Solstice's left nav) that opens each of
Abdullah's tools inside the same window, so the team has one link instead of three:
Proof Coin Dashboard, Stock Take Dashboard, SLI Generator.

## How it works (deliberately simple)
`index.html` is a single file. The side menu and home tiles are generated from the `APPS`
list at the top of the `<script>`. Choosing a tool loads its live URL into an `<iframe>`;
the apps themselves are untouched. Remembers last tool + menu state per browser.

**To add a new app:** add one line to `APPS` (id, icon, label, url, blurb). Nothing else.

Pre-check done 2026-09-11: none of the three apps send X-Frame-Options / CSP frame-ancestors,
so embedding works. If a future app refuses to embed, the "Open in new tab" link in the
top bar is the fallback.

## Current state (2026-09-11)
- v1 built and verified locally in Chrome (home tiles, side menu, SLI loads inside frame,
  Proof Dashboard login form loads inside frame).
- NOT yet hosted. Next step: put it on GitHub Pages (same as SLI Generator) so the team can
  use one link. GitHub Pages has no usage quota, unlike Vercel Hobby.
- Proof Coin Dashboard login works again as of 2026-09-11 (project moved to the Bullion Boeties
  Vercel team with a fresh store, 0 coins, staff logins recreated — see Proof Dashboard project).
- Stock Take tile still points at stock-take-dashboard.vercel.app, which is the OLD personal
  project until Abdullah moves that domain to the new Bullion Boeties project (open item there).

## Key files
- `Projects\Inventory Hub\index.html` — the whole app.

## Pending / ideas
- Host on GitHub Pages, share link with Zubayr/Angie.
- Later (only if wanted): shared single login across Proof + Stock Take (both already use
  the same name+PIN signed-token pattern).
- Could become the shell for the personal Daily Desk plan too.
