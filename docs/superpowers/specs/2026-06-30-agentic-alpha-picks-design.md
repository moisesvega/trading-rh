# Design — Agentic Robinhood portfolio backtester + executor

- **Date:** 2026-06-30
- **Status:** draft for review

## Purpose

A small, "totally agentic" workspace where **Claude Code is the agent** and drives the
connected **Robinhood MCP** directly — no standalone app or API code. It ingests
portfolio definitions (e.g. Seeking Alpha *Alpha Picks*) from CSV, **backtests** them
against historical prices to find the best performer, and can **autonomously execute**
the chosen portfolio on a designated cash account within a hard risk envelope.

## Decisions (locked)

| Area | Decision |
|---|---|
| Agent shape | Claude Code + Robinhood MCP; capabilities as skills. No app/API/SDK code. |
| Compute | Pure agentic (no product code). Backtest math done in-context, but every run **writes full numbers + assumptions to a report file** and includes a SPY self-check. |
| Trading level | Fully autonomous execution, bounded by hard guardrails. |
| Strategy model | Rule-based, **file-driven**: the input CSV is the source of truth. Mirror the pick list (buy new, sell dropped). |
| Instruments | Equities/ETFs only. |
| Live account | **Agentic cash account `821056652`** only (`agentic_allowed=true`). Margin account is not agent-reachable. |
| Sizing | Equal-weight = budget ÷ active picks, ceiling $500/name. |
| Backtest defaults | Equal-weight, monthly rebalance, vs SPY, $10k start, metrics: total return / CAGR / max DD / win rate. |

## Hard guardrails (see `config/settings.yaml`)

- **`never_trade: [UBER]`** — permanent; enforced before every order and on every import.
- Total capital cap **$2,000** (subject to real buying power), max **$500**/position.
- **Limit** orders at/near ask; **regular hours only**; never market/extended
  — *unless* `allow_fractional_market` is explicitly enabled (fractional shares require market orders).
- Reconcile **daily** after open. **Kill switch**: on data anomaly or large drift, halt + report, don't trade.

## Repo layout

```
CLAUDE.md                 operating instructions + guardrails for the agent
config/settings.yaml      risk envelope, account, backtest defaults
portfolios/*.csv          INPUT — one portfolio per file
data/*.json               cached price history pulled from the MCP
backtests/*.md            OUTPUT — one report per backtest run
journal/*.md              OUTPUT — live trade log + reasoning (one file per day)
.claude/skills/           backtest / compare-portfolios / execute   (to be built)
```

### Portfolio CSV format
```
symbol,quant_rating,ref_price,date_added,status
BRK.B,3.45,500.39,2026-06-30,hold        # status: hold | sold ; drives buy/sell reconcile
```

## Section 3 — Backtest engine (flow)

1. Read portfolio CSV → active symbols (`status=hold`, minus `never_trade`).
2. Pull monthly OHLCV per symbol + SPY via `get_equity_historicals`; cache to `data/`.
3. Simulate equal-weight over the window (buy&hold and/or monthly-rebalanced).
4. Metrics: total return, CAGR, max drawdown, win rate; benchmark vs SPY.
5. Write `backtests/<date>-<portfolio>.md` with per-name numbers, assumptions, and the
   **SPY self-check** (backtested SPY must match SPY's actual move).
6. **Always disclose** survivorship/lookahead bias for snapshot portfolios (no add/sell dates).

## Section 4 — Live execution engine (flow)

1. Read target portfolio + `config`. Confirm account `821056652`, get buying power + positions.
2. Diff: buy `hold` names not held; sell held names that are `sold`/absent.
3. Enforce guardrails (never_trade, caps, tradability, session). Size = min(budget/N, $500).
4. `review_equity_order` → present → `place_equity_order` (limit near ask). Log all to `journal/`.
5. Kill switch on anomalies. Autonomy trigger (daily) to be wired via a scheduled agent.

## Section 5 — Error handling & testing

- Missing/failed historicals → skip symbol in backtest (note it); in live, **halt** that name, never guess.
- Order rejected → log, surface, no blind retry.
- **Dry-run mode**: compute intended orders without placing (used before any real execution).
- Backtest validation: the SPY self-check catches math drift.

## Known open items

- **Fractional vs limit conflict:** $1,000 equal-weight over 41 names ≈ $24/name → needs
  fractional shares (market orders). Resolve before enabling live execution.
- **Bias:** snapshot portfolios yield inflated backtests; a trustworthy forward test needs
  point-in-time pick history.
- **Autonomy mechanism:** choose scheduled cloud agent vs cron for the daily trigger.

## Not building (YAGNI)

Options, crypto, futures; multi-account; standalone app/API/SDK; XLSX ingestion (convert to CSV).
