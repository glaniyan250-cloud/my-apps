# LOG Investments Screener — S&P 500 bulk data

This folder's bulk file loads the whole S&P 500 into the screener in one
import, instead of adding companies one by one.

**File:** `log-investments-screener-sp500.json`
**Built:** 31 October 2026, from the Wikipedia S&P 500 constituent list
fetched that day (503 tickers — 500 companies, some with two share
classes).
**Result on import:** 503 companies — 323 pass the screener's Stage 1
gates and appear in Ranked, 180 go to Excluded with the reason stated.

## How to use it

1. Open the screener: `log-investments-screener.html`
   (or the same file on the my-apps hub).
2. **Export first if you already have manual companies in the screener
   that you want to keep.** Import replaces all current data.
3. Click **Import JSON**.
4. Choose `log-investments-screener-sp500.json`.
5. Confirm **Replace** when asked.
6. Use the **Ranked** tab for scores and verdicts, the **Excluded** tab
   for companies that failed a gate (with the reason), and **Detail** on
   any company to see why it passed or failed each metric.
7. Star companies to the watchlist and save a quarterly snapshot as
   usual. The file sets Treasury to 4.5% and minimum history to 5 years;
   benchmarks and weights fall back to the screener's defaults.

## What the prices are

EDGAR publishes no prices, so no price in this file comes from EDGAR.

- **Price date for every company in this run: 2026-10-30** (the last
  trading day before this build). Each company also carries
  `price_date` and `price_source` fields, so you can check this in the
  file itself.
- **Price source in this run: Yahoo Finance for all 503.** The script
  tries Stooq first, as designed, but Stooq currently blocks script
  requests behind a JavaScript check, so every price fell through to
  the Yahoo fallback. No price was invented: a company with no price
  would have been left blank and landed in Excluded.
- Prices will be out of date the next trading day. Re-run for fresh
  prices (see below) rather than treating these as current.

## What the capex figure really is

**Capex in this file is total PPE capex, used as the maintenance-capex
proxy — not a separately reported maintenance capex figure.**

The script uses `PaymentsToAcquirePropertyPlantAndEquipment`, with
`PaymentsToAcquireProductiveAssets` as a fallback for filers that use
that tag instead (for example A. O. Smith). Companies that do not
report either tag — many insurers, for example — have capex left blank
and land in Excluded. That is honest, not a gap to fill by guessing.

Two other parsing notes, so the numbers are not a mystery:

- Flow figures (revenue, cash flow, capex, net income, depreciation)
  keep full-year points only, so a quarterly breakdown inside a 10-K
  is never mistaken for a full year. The revenue tag with the most
  recent year wins, rather than the first tag in a fixed list.
- Share counts are read from "shares" units (the screener's one-ticker
  fetch only reads USD units, which is why it always missed shares).
  Where a filer's share tag stops more than two years before its
  latest revenue year, the share figure is dropped rather than paired
  with a current price. A. O. Smith is one such case.

## Why 180 companies are excluded

Top missing fields (a company can miss more than one):

- Maintenance capex missing: 70
- Shares outstanding missing: 53
- Net income missing: 37
- Equity missing: 29
- Revenue missing: 15
- Operating cash flow missing: 9
- Fewer than 5 years of revenue history: 32 in total, including
  15 with 0 years

Excluded does not mean "bad company". It means the free EDGAR tags
used here did not supply a complete, current set of figures for that
company. Berkshire Hathaway (BRK.B) is one example: it is in the file
with a price, but with no point-in-time share-count tag, so it lands
in Excluded. If you want a specific excluded company ranked, enter its
missing figures manually from its own filings and it will move to
Ranked.

Multi-class note: GOOG and GOOGL share one SEC filing, so their
fundamentals are identical in this file and their scores differ only
by share price. Treat them as one company, not two independent passes.

## First pass, not diligence

Free data has delays and gaps, and this bulk load inherits them at
scale. Use it to shortlist, then verify every figure that matters
against the company's own filings before any decision. This is a
learning and screening tool, not investment advice.

## How to re-run it (quarterly, after earnings season)

The screener's own rhythm is quarterly after earnings season
(mid-February, mid-May, mid-August, mid-November) and on demand.

```bash
python3 build-bulk-edgar.py --refresh --refresh-prices
```

Without `--refresh` / `--refresh-prices`, the script reuses its cache
under `~/workspace/log-investments-bulk-work/cache/`, so an interrupted
run can be resumed. For a quarterly refresh you want fresh filings
and fresh prices, so use both flags, then import the new JSON file
into the screener again.

SEC note: the script's default User-Agent uses
`contact@example.com`, a non-personal placeholder, not a real mailbox
and not anyone's private address. SEC's fair-access guidance asks for
a contact address, so replace it with your own contact — or set the
`LOG_EDGAR_USER_AGENT` environment variable — before repeated heavy
use. The script never stores a personal email in its output.
