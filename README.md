# Ike Today

Today's money in at MRH Investment, split into **Cash** and **Transfer** —
two cards, each topped by its own total, listing every payment as
`invoice number · customer · amount`. Like its siblings this repo is
**static only**: no secret, no cron, no build step. It fetches its data live
from [ike-data](https://github.com/yuki-uthman/ike-data), the shared Odoo
pipeline that also backs [ike-sales](https://github.com/yuki-uthman/ike-sales)
and ike-expenses.

## What it counts, and what it does not

This page counts **money received today**, not sales made today — the two are
different numbers and both are right:

- A credit sale invoiced today but settled next week shows up in **ike-sales
  today** and here **next week**.
- A customer clearing last month's invoice at the counter today shows up
  **here today** and in ike-sales **last month**.

"Today" is the Maldives day (UTC+5), the same boundary `fetch_sales.py` uses,
so the two dashboards always agree on which day it is.

## Cash vs transfer

The split comes from the name Odoo gives each payment method (POS) or journal
(accounting): anything matching `cash` is cash, anything matching
`transfer|bank|bml|mib|mfl|online|deposit` is a transfer. Anything else —
a cheque, a card, a "Customer Account" credit line — is **not** silently
counted as a transfer. It lands in an `other` bucket that renders as an amber
strip under the two cards, naming the method Odoo actually used.

That strip is a prompt, not an error: when it appears, widen the patterns in
`ike-data/scripts/fetch_today.py` and it stops appearing. The run log of every
refresh prints each distinct method name it saw, so you never have to guess
what the real names are.

## One-time setup

**Enable GitHub Pages**: Settings → Pages → Source: "Deploy from a branch" →
Branch: `main`, folder `/ (root)`. The page will be at
`https://<your-username>.github.io/ike-today/`.

Nothing else lives here. The Odoo credential, the schedule, and the
cash/transfer classification are all in
[ike-data](https://github.com/yuki-uthman/ike-data).

## How it gets its data

`index.html` fetches
`https://raw.githubusercontent.com/yuki-uthman/ike-data/main/data/today.json`
in the browser on every page load (`cache: 'no-store'`), then silently again
every 60 seconds and whenever you switch back to the tab — the underlying file
itself refreshes every 15 minutes. GitHub serves raw file content with
`Access-Control-Allow-Origin: *`, so this works cross-repo with no server or
API needed.

## Visibility

GitHub Pages on a free plan requires a **public** repository, so this page is
technically reachable by anyone with the exact URL, though it isn't linked or
indexed anywhere. Note this dashboard shows **customer names and invoice
numbers**, which the sales and expenses dashboards do not. If that matters,
GitHub Pages on private repos requires GitHub Pro or higher.
