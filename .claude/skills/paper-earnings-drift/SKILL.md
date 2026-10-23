---
name: paper-earnings-drift
description: Use when asked to run, update, or report on the earnings paper trade, scan recent earnings for post-earnings-drift entries, or check how the paper earnings strategy is doing versus SPY.
---

# Paper-trade post-earnings drift

## Overview

Forward-test one rule before any money touches it: **buy stocks the market rewarded for a
beat, after the report, and hold for the drift.** Everything is simulated in
`paper/earnings-drift.csv` at closing prices. Parameters live in
`config/settings.yaml` → `paper_earnings_drift`.

## Hard rules

- **PAPER ONLY. Never call `place_*`, `review_*`, or `cancel_*` order tools from this skill.**
  Read-only tools only (calendar, earnings results, quotes, historicals).
- Forward-only: never add an entry whose `entry_date` is before `started` or before today's
  run. Backfilling past entries leaks known outcomes and makes the test meaningless.
- `never_trade` (UBER) applies here too — skip it.
- Never edit a closed row. The ledger is append/close-only.
- Today's daily bar is usually still `interpolated` at run time. For **today only**, use the
  day-1 fallback: close = `get_equity_quotes` official close for today, open = open of today's
  first `30minute` bar. Note `fallback` on the journal line. Any other gap (missing bar, a past
  bar `interpolated`, fallback data missing, missing EPS actual) → skip that name and note it
  in the journal. Do not guess prices.

## Procedure (run once per trading day, after the close)

1. **Load** `config/settings.yaml` and `paper/earnings-drift.csv`.
2. **Close open rows first.** For each `status=open` row, get daily closes since entry
   (`get_equity_historicals`, `interval: day`). Walk forward day by day; the first hit wins:
   - close ≤ `stop_price` → close at that day's close, `exit_reason=stop`;
   - `exit_due` reached → close at that day's close, `exit_reason=time`.
   Fill `exit_date, exit_price, ret_pct`, SPY close on exit → `spy_exit, spy_ret_pct`,
   `status=closed`.
3. **Find candidates.** `get_earnings_calendar` with `filter: high_market_cap`, `days: -3`.
   Day 1 = the first regular session that trades on the news: the report date for `am`
   timing, the next session for `pm`. Keep only reports whose day 1 is **today**.
4. **Apply the rule** to each candidate (`get_earnings_results` + daily bars):
   - EPS actual > estimate (if `require_eps_beat`);
   - `gap_pct` = day-1 close / pre-report close − 1 ≥ `min_gap_pct`;
   - day-1 close ≥ day-1 open (if `require_hold_gap`).
5. **Enter** passing names, biggest `gap_pct` first, until `max_open` open rows:
   `entry_price` = day-1 close, `stop_price` = entry × (1 + `stop_loss_pct`/100),
   `exit_due` = entry + `hold_trading_days` trading days, `spy_entry` = SPY day-1 close,
   `status=open`.
6. **Journal** in `journal/<today>.md` under `## Paper: earnings drift`: candidates seen, which
   passed/failed and why, entries, exits, and running totals —
   closed-trade count, win rate, average `ret_pct` vs average `spy_ret_pct`, and the
   equal-weight paper P&L at `position_usd` per trade.

## Review (on or after `review_after`)

Report: trades, win rate, average return vs SPY over the same windows, worst trade, and how
many stops hit. Fewer than ~15 closed trades is not enough to conclude anything — say so.
Going live is the user's decision; it needs its own guardrails in `config/settings.yaml`
and the `execute-portfolio` review flow. This skill never trades.
