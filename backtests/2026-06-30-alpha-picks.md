# Backtest — Alpha Picks basket (UBER excluded)

- **Run date:** 2026-06-30
- **Portfolio:** `portfolios/alpha-picks.csv` — 41 names (UBER excluded per hard rule)
- **Window:** 2025-06-01 (first monthly open) → 2026-05 (last monthly close), ~12 months
- **Method:** equal-dollar, buy-and-hold. Monthly-bar granularity. Price return (no dividends).
- **Benchmark:** SPY

## Headline

| Metric | 41-name basket | SPY |
|---|---|---|
| Total return | **+282%** | +29% |
| CAGR (~1yr) | +282% | — |
| Max drawdown (monthly) | −4.1% | −6.0% |
| Win rate | 36/41 = 88% positive | — |

**Median per-name return: +108%.** Mean per-name return: +282%.

## ⚠️ Why this number is not trustworthy as a forward signal

1. **Survivorship / lookahead bias (the big one).** These 41 names are the *current* top quant-rated stocks. Backtesting them backward asks "what if I'd bought a year ago the names that a year later turned out to be winners." Of course it looks spectacular. This is hindsight, not a repeatable edge.
2. **One name drives a third of it.** SNDK returned **+4,415%** ($37.54 → $1,695) and alone contributes **+108 of the +282 pts**. Cap each name at +300% and the basket return drops to **+129%** — still huge, but shows how outlier-dependent the mean is.
3. **No dividends, monthly granularity.** Drawdown is understated (monthly bars miss intra-month lows); returns exclude dividends.
4. **No entry/exit history.** The snapshot has no `date_added`/`date_sold`, so this is a static basket, not a true point-in-time Alpha Picks replay.

## Dispersion

- **Top 5:** SNDK +4415%, LITE +1071%, MU +927%, MXL +714%, TTMI +484%
- **Bottom 5:** TMUS −22%, EAT −17%, CVSA −10%, BRK.B −5%, ALL −1%
- 23 of 41 names >+100%; only 5 negative.

## Honest takeaway

Over the last 12 months this basket *would have* crushed SPY (+282% vs +29%) with a smaller drawdown — but that is almost entirely **selection hindsight**, concentrated in a handful of AI/memory names (SNDK, MU, LITE, MXL). It says these were great stocks to own a year ago; it does **not** establish they'll outperform going forward. A credible test needs point-in-time pick data (real `date_added`/`date_sold`) to remove lookahead bias.

_Self-check: SPY backtested here = +28.7% (587.76 → 756.48), consistent with SPY's actual move — math path validated._
