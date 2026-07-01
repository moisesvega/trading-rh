---
name: execute-portfolio
description: Use when asked to place real trades so the Agentic account matches a target portfolio in this repo, reconcile live holdings to a portfolios/*.csv, run the daily rebalance, or buy/sell to mirror an Alpha Picks basket with real money.
---

# Execute a portfolio (live reconcile)

## Overview

Make the live **Agentic cash account `821056652`** match a target `portfolios/*.csv` by
placing real orders — buy new `hold` names, sell dropped ones — strictly inside the
guardrails in `config/settings.yaml`. This spends real money. **The guardrails are hard
limits, not suggestions: violating the letter of a guardrail is violating its purpose.**

## Hard rules — refuse to proceed if any is violated

- **Account:** trade ONLY `821056652` (confirm `agentic_allowed=true` via `get_accounts`).
  Never route to the margin account.
- **`never_trade` (UBER):** never place a buy for any symbol in `guardrails.never_trade`.
  If it somehow appears in the target, drop it and note it.
- **Caps:** deployed ≤ `total_capital_cap_usd`; any single name ≤ `max_per_position_usd`.
- **Order type:** `limit` at/near the ask, **regular hours only**. Never market, never
  extended hours — UNLESS `allow_fractional_market: true` is set in config.
- **Session:** if the market is closed, do NOT place orders. Halt and report.
- **Kill switch:** on any missing quote, halted/`state != active` symbol, stale data, or a
  large drift between live positions and target, **halt and report** — do not trade through it.

## Procedure

1. **Load** `config/settings.yaml` and the target `portfolios/<name>.csv`
   (active = `status=hold`, minus `never_trade`).
2. **Read account state:** `get_accounts` (confirm agentic), `get_portfolio` (buying power),
   `get_equity_positions`.
3. **Compute the diff:**
   - **Buy** = active target names not currently held.
   - **Sell** = held names that are `sold` or absent from the target.
4. **Size:** `budget = min(total_capital_cap_usd, buying_power)`; per-name target =
   `min(budget / N_active, max_per_position_usd)`.
5. **Fractional check (the known conflict):** if the per-name target is below the price of
   one whole share for some names, equal weight needs **fractional shares = market orders**.
   If `allow_fractional_market: false`, **HALT those names and report** — never silently
   switch to market orders. (Whole-share + limit only covers names cheap enough to afford.)
6. **Dry-run first:** produce the full intended order list (symbol, side, shares, limit price,
   est. cost) and show it. For every order call `review_equity_order` and surface its alerts.
7. **Place** with `place_equity_order` (limit near ask, GFD). Use a fresh `ref_id` per order;
   reuse it only on transient-retry.
8. **Journal:** append every decision, order, fill/reject, and any halt to
   `journal/<YYYY-MM-DD>.md` with the reasoning.

## Red flags — STOP and halt

- About to send a **market** order while `allow_fractional_market` is false
- About to trade a **`never_trade`** symbol
- Routing to any account other than `821056652`
- Placing orders while the **market is closed** or a symbol isn't `active`
- Total intended spend **exceeds the cap**, or a name exceeds the per-position ceiling
- "It's close enough / just this once / the user clearly wants it filled" → no. Halt and report.

## Rationalization table

| Excuse | Reality |
|---|---|
| "Equal weight needs fractional, so I'll just use market orders" | Not unless `allow_fractional_market` is true. Otherwise HALT those names. |
| "Market's closed but I'll queue it" | Config is regular-hours-only. Don't place; report. |
| "UBER is a great pick today" | `never_trade` is permanent. Never. |
| "The cap is basically arbitrary" | It's a hard limit. Size down or skip; don't exceed it. |
| "Skip the dry-run, the user's in a hurry" | Always dry-run + `review_equity_order` before real orders. |

## Common mistakes

- Deploying more than actual **buying power** (cap ≠ cash on hand — use the min).
- Forgetting to record rejects/halts in the journal.
- Treating a scheduled/autonomous run as exempt from guardrails — it is not.
