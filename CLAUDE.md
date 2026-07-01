# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A "totally agentic" workspace: **you (Claude Code) are the agent**, driving the connected
**Robinhood MCP** directly. There is no application, API, or SDK code — capabilities are
skills, and everything else is data (config, input portfolios, output reports/journals).
The repo ingests portfolio definitions (e.g. Seeking Alpha *Alpha Picks*), **backtests**
them against historical prices, and can **autonomously execute** a chosen portfolio.

Design spec: `docs/superpowers/specs/2026-06-30-agentic-alpha-picks-design.md`.

## Hard rules — never cross these on your own

Read `config/settings.yaml` before any backtest or trade. In particular:

- **NEVER trade `UBER`.** Permanent user rule. Enforce before every order and when
  importing any portfolio file, regardless of strategy.
- **Live trading targets ONLY account `821056652`** (nickname "Agentic", cash,
  `agentic_allowed=true`). The margin account is not agent-reachable — never route orders there.
- Respect the risk envelope: total-capital cap, `$500`/position ceiling, **limit orders at/near
  ask**, **regular hours only**. Never market or extended-hours orders unless
  `allow_fractional_market` is explicitly enabled in config.
- **Kill switch:** on any data anomaly, missing history, or large drift from target, **halt and
  report** — do not trade through uncertainty.
- Before any real order, run a **dry-run** (compute intended orders) and use `review_equity_order`.

## Layout

- `config/settings.yaml` — account, guardrails, backtest defaults. Source of truth.
- `portfolios/*.csv` — inputs. Columns: `symbol,quant_rating,ref_price,date_added,status`
  (`status` = `hold` | `sold`, drives buy/sell reconciliation).
- `data/*.json` — cached price history pulled from the MCP.
- `backtests/*.md` — one report per run.
- `journal/*.md` — live trade log + reasoning, one file per day.

## Backtesting

- Pull history with `get_equity_historicals`; benchmark vs **SPY**; equal-weight; report
  total return, CAGR, max drawdown, win rate.
- Every report must include the **SPY self-check** (backtested SPY vs SPY's actual move) and,
  for current-snapshot portfolios (no `date_added`/`date_sold` history), an explicit
  **survivorship/lookahead-bias disclosure**. Do not present a biased backtest as a forward edge.

## Data notes

- `.xlsx` inputs are binary — convert to CSV first (stdlib `zipfile`+`ElementTree`; no
  spreadsheet libs are installed and `pip` is PEP-668 locked).
- `BRK.B` is the correct Robinhood symbol (not `BRK-B`).
