# Backtest variations — Alpha Picks basket (UBER excluded)

- **Run date:** 2026-06-30 · **Window:** 2025-06-01 → 2026-05 (~12 mo) · **Data:** `data/alpha-picks-monthly.json`

| Variant | Total return | Max DD (monthly) |
|---|---:|---:|
| Equal-wt **buy & hold** (baseline) | **+282%** | −4.1% |
| Equal-wt **monthly-rebalanced** | +181% | −3.5% |
| Buy & hold, **ex-SNDK** (drop top outlier) | +178% | −4.9% |
| Buy & hold, **ex-top-5 winners** | +109% | −4.8% |
| Buy & hold, **each name capped +300%** | +129% | — |
| **SPY benchmark** | +29% | −6.0% |

Median name +108% · winners 36/41 · 23 names >+100%.

## Reading the variations

- **Removing outliers doesn't rescue the conclusion.** Drop SNDK → still +178%. Drop the top 5 → still **+109%**, ~4× SPY. The outperformance isn't one lucky stock; it's that **all 41 are hindsight-selected survivors**. This is the survivorship/lookahead bias made visible.
- **Rebalancing trims winners.** Monthly-rebalanced (+181%) trails buy-and-hold (+282%) because it sells the moon-shots down to equal weight each month. In a mania that costs you; in a normal regime it controls risk.
- **Every variant beats SPY by a wide margin** — which is exactly what a biased backtest looks like. A clean, forward-looking test still requires point-in-time pick history (`date_added`/`date_sold`).

_These numbers describe the past of a set chosen with knowledge of that past. They are not a forward edge._
