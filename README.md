# Ike POS

The till, day by day, at MRH Investment: money **in**, split into **Cash**
and **Transfer** cards, plus what got taken back **out** of the drawer in a
third **Cash Out** card. A date pill strip across the top (same pattern as
[ike-sales](https://github.com/yuki-uthman/ike-sales), minus the chart) lets
you flip back through the retained history instead of only ever seeing
today. Like its siblings this repo is
**static only**: no secret, no cron, no build step. It fetches its data live
from [ike-data](https://github.com/yuki-uthman/ike-data), the shared Odoo
pipeline that also backs ike-sales and ike-expenses.

## What it counts, and what it does not

**Cash** and **Transfer** count **money received that day**, not sales made
that day — the two are different numbers and both are right:

- A credit sale invoiced today but settled next week shows up in **ike-sales
  today** and here **next week**.
- A customer clearing last month's invoice at the counter today shows up
  **here today** and in ike-sales **last month**.

"Today" is the Maldives day (UTC+5), the same boundary `fetch_sales.py` uses,
so the two dashboards always agree on which day it is.

**Cash Out** is a different, narrower question: the POS register's own
"Cash Out" button — a staff member taking money out of the till for a float
pickup, petrol, a gate pass, and so on. These have no `hr.expense` or
`account.payment` behind them at all, so this is the only place they're
visible; confirmed expenses staff filed through the Expenses app live on the
separate **ike-expenses** dashboard instead. Cash Out is *not* net against
Cash — the two cards are independent totals of what came in and what went
out, not a running balance.

## Cash vs transfer

The split comes from the name Odoo gives each payment method (POS) or journal
(accounting): anything matching `cash` is cash, anything matching
`transfer|bank|bml|mib|mfl|online|deposit` is a transfer. Anything else —
a cheque, a card, a "Customer Account" credit line — is **not** silently
counted as a transfer. It lands in an `other` bucket that renders as an amber
strip under the cards, naming the method Odoo actually used.

That strip is a prompt, not an error: when it appears, widen the patterns in
`ike-data/scripts/odoo_payments.py` and it stops appearing. The run log of
every refresh prints each distinct method name it saw, so you never have to
guess what the real names are.

## The expanded rows

A Cash/Transfer row expands only when ike-data found lines behind it — the
POS order's lines for a counter sale, the reconciled invoice's lines for a
bank receipt. Line totals are tax-inclusive, the same basis as the row amount
and the card total above it, so for a payment that settles an invoice in full
the lines add up to the row. A partial payment expands to the whole invoice's
lines, which will therefore total more than the payment itself. Cash Out rows
don't expand — there's no product detail behind a till withdrawal, just the
reason staff typed in at the register.

Rows stay open across the 60-second refresh — an open row is remembered by
its reference and amount, not its position, so a new payment arriving does
not slam shut the row you were reading.

## The date pills

Picking a pill shows that day's cards from the same retained history
ike-sales charts (60 days). Only "Today"'s card header shows the live
"updated HH:MM" stamp; a past day is a closed, settled snapshot. A 60-second
refresh updates the pill labels/totals in place without resetting your
scroll position or which day you're looking at.

## One-time setup

**Enable GitHub Pages**: Settings → Pages → Source: "Deploy from a branch" →
Branch: `main`, folder `/ (root)`. The page will be at
`https://<your-username>.github.io/ike-pos/`.

Nothing else lives here. The Odoo credential, the schedule, and the
cash/transfer/cash-out classification are all in
[ike-data](https://github.com/yuki-uthman/ike-data).

## How it gets its data

`index.html` fetches
`https://raw.githubusercontent.com/yuki-uthman/ike-data/main/data/sales.json`
directly in the browser on every page load (`cache: 'no-store'`) — the same
60-day history file ike-sales reads, so the two dashboards can never disagree
about a day's numbers. It refetches silently every 60 seconds and whenever
you switch back to the tab; the underlying file itself refreshes every 15
minutes. GitHub serves raw file content with `Access-Control-Allow-Origin:
*`, so this works cross-repo with no server or API needed.

## Visibility

GitHub Pages on a free plan requires a **public** repository, so this page is
technically reachable by anyone with the exact URL, though it isn't linked or
indexed anywhere. Note this dashboard shows **customer names, invoice
numbers, and the reasons staff give for a till cash-out**, which the sales
and expenses dashboards do not. If that matters, GitHub Pages on private
repos requires GitHub Pro or higher.
