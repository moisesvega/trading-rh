---
name: compare-portfolios
description: Use when asked to rank or compare multiple portfolios or strategies in this repo by backtested performance, decide which basket is best, or produce a leaderboard across portfolios/*.csv.
---

# Compare portfolios

## Overview

Rank several `portfolios/*.csv` baskets against each other over a **common window** so the
comparison is fair, and produce a leaderboard. Reuses the `backtest-portfolio` skill for
each basket rather than reimplementing the math.

**REQUIRED SUB-SKILL:** use `backtest-portfolio` to produce each portfolio's metrics.

## Procedure

1. **Pick the set** — all `portfolios/*.csv`, or the ones the user named.
2. **Fix one window** for every portfolio (same start/end, same benchmark SPY, same
   weighting). A comparison across different windows is meaningless — state the window.
3. **Backtest each** via the `backtest-portfolio` skill. Reuse `data/*.json` caches.
4. **Rank** by total return **and** a risk-adjusted view (return ÷ |max drawdown|), plus the
   **median-name** return so an outlier-driven basket doesn't win on one moonshot alone.
5. **Write** `backtests/<YYYY-MM-DD>-comparison.md`: one row per portfolio (total return,
   CAGR, max DD, win rate, median name), SPY as the baseline row, and the winner called out.
6. **Carry the bias caveat forward.** If the ranked baskets are snapshots, the ranking
   reflects past selection, not a forward edge — say so.

## Quick reference

| Inputs | 2+ `portfolios/*.csv` |
| Fairness rule | identical window + benchmark for all |
| Rank keys | total return, return/|maxDD|, median-name return |
| Output | `backtests/<date>-comparison.md` |

## Common mistakes

- **Comparing across different windows.** Always align dates first.
- **Ranking on mean total return only.** An outlier can top the table; show median + risk-adjusted.
- **Re-deriving backtest math here.** Delegate to `backtest-portfolio`.
