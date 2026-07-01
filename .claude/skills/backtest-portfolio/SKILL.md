---
name: backtest-portfolio
description: Use when asked to backtest a portfolio in this repo, measure how a portfolios/*.csv basket (e.g. Alpha Picks) would have performed historically, or compare a strategy's return and drawdown against SPY.
---

# Backtest a portfolio

## Overview

Replay a `portfolios/*.csv` basket over history using Robinhood MCP price data, and
report total return, CAGR, max drawdown, and win rate versus SPY. Compute is done with
stdlib Python (deterministic), and every run writes a full report to `backtests/`.

## Procedure

1. **Read config.** `config/settings.yaml` → `guardrails.never_trade`, `backtest.*`
   (benchmark, weighting, rebalance, starting_capital). Never include a `never_trade`
   symbol in the simulated basket.
2. **Read the portfolio.** Active names = rows with `status = hold`, minus `never_trade`.
3. **Pull history.** `get_equity_historicals` for the names **plus SPY**, `interval: month`
   (weekly/daily only if a finer drawdown is requested). Batch ≤10 symbols per call.
   Cache the parsed `{start_open, monthly_closes}` to `data/<portfolio>-monthly.json` so
   reruns don't refetch.
4. **Simulate:**
   - **Weighting matters more than anything.** If the CSV has a `holding_weight_pct` column
     (a real portfolio like Alpha Picks), weight by it — do NOT equal-weight. Equal weight
     over-counts tiny recent moonshots (e.g. a 0.5% position up +800%) and produces nonsense.
   - **For a YTD/period return, use BEGINNING-of-period weights**, not current weights.
     Current holding % already contains this period's gains, so weighting the period's returns
     by current weights double-counts winners (seen: +82% vs the correct +46%). Recover a
     start weight as `current_w / (1 + period_return)`. For positions **added mid-period**,
     use their since-add return, not a full-period return.
   - Entry = first bar's open; exit = last bar's close (or today's quote for "to today").
   - Equal-weight buy-and-hold path: `V_t = mean_i(price_i,t / start_i)`; report
     monthly-rebalanced too when comparing.
5. **Metrics:** total return, CAGR, max drawdown (over the value path), win rate. Benchmark
   the same window for SPY.
6. **SPY self-check.** Backtested SPY total return must match SPY's actual start→end move.
   If it doesn't, the math is wrong — stop and fix before reporting.
7. **Bias disclosure (required).** If the portfolio is a current snapshot (no real
   `date_added`/`date_sold` history), state prominently that results carry
   **survivorship/lookahead bias** and are not a forward edge. Show the median-name return
   and an outlier-capped figure so one moonshot can't masquerade as skill.
8. **Write** `backtests/<YYYY-MM-DD>-<portfolio>.md`: headline table, per-name dispersion,
   assumptions, the SPY self-check line, and the bias note.

## Quick reference

| Input | `portfolios/<name>.csv` |
| Benchmark | SPY (always) |
| Weighting | equal-dollar; report buy&hold and monthly-rebalanced |
| Granularity | monthly bars (default) |
| Output | `backtests/<date>-<name>.md` + `data/<name>-monthly.json` |

## Common mistakes

- **Presenting the raw total return as a forward signal.** For snapshots it's biased; always caveat.
- **Skipping the SPY self-check.** It's the only guard against silent math errors.
- **Letting one outlier define the result.** Report median + capped alongside the mean.
- **Refetching each run.** Reuse the `data/` cache.
- **Including `never_trade` names.** Filter them out before simulating.
