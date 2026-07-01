# Backtest — Alpha Picks, REAL holding weights (YTD 2026)

- **Run date:** 2026-06-30 · **Source:** real Alpha Picks holdings → `portfolios/alpha-picks-holdings.csv`
- **48 positions** (some symbols held as multiple lots), weights sum to 100%.

## The answer: use real weights, at start-of-year

| Method | YTD 2026 | Verdict |
|---|---:|---|
| Equal weight | +79% | ❌ over-weights tiny recent moonshots |
| Real weights, **current** | +82% | ❌ over-counts names that already soared this year |
| Real weights, **start-of-year** (with UBER) | **+46.1%** | ✅ matches the reported ~42% (gap = monthly-close prices + snapshot timing + mid-year cash) |
| Real weights, **start-of-year, UBER removed** | **+48.1%** | tradable version — never-trade rule; UBER was a −12% YTD drag |
| SPY | +9.5% | benchmark |

## UBER removed (tradable portfolio)

Per the permanent `never_trade: [UBER]` rule, the actionable portfolio drops UBER (was 2.06%,
−12% YTD) and renormalizes the remaining 47 positions to 100% (scale ×1.0211). YTD improves to
**+48.1%**. See `portfolios/alpha-picks-holdings-no-uber.csv`. The full 48-position file
(`alpha-picks-holdings.csv`) is kept for analysis/reference only — it reflects the real Alpha
Picks portfolio, which does include UBER.

## Why equal weight was so wrong

Alpha Picks lets winners run, so **weight ≈ accumulated appreciation**. The heavyweights are the
old compounders; the headline-grabbing recent rockets are tiny:

| Name | Picked | Weight | 2026 YTD |
|---|---|---:|---:|
| STRL | 2023-08 | 10.67% | +174% |
| POWL | 2023-05 | 10.65% | +170% |
| AGX | 2024-10 | 5.03% | +155% |
| MU | 2025-10 | 3.65% | +303% |
| **SNDK** | **2026-06** | **0.47%** | +857% |
| MXL | 2026-05 | 0.42% | +635% |

SNDK's +857% is nearly irrelevant at 0.47% weight — but equal weight (1/42 ≈ 2.4%) gave it 5×
its real influence, which is how the equal-weight backtest ballooned to +282%/+351%.

## Methodology note (important for the skill)

A YTD/period return must weight by **beginning-of-period** weights. Current holding % already
bakes in this year's gains, so weighting YTD returns by current weights double-counts winners
(+82% vs the correct +46%). For positions **added mid-year**, use their since-add return, not a
full-year return.

_Prices are monthly-close approximations; SA's official figure uses exact daily closes, hence the
~46% vs ~42% gap._
