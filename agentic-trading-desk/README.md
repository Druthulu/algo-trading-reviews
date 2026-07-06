# Agentic Trading Desk — independent backtest review

An independent, pre-registered backtest of [**Agentic Trading Desk**](https://github.com/Oft3r/agentic-trading-desk)
(Oft3r) — the Claude-skill for daily technical analysis on stocks/ETFs.

**Verdict: no timing edge.** Mechanized faithfully (the author's three scripts,
byte-for-byte) and driven over ~9.4 years of daily bars (2017-02-27 → 2026-06-18),
the system is beaten by a plain 200-day moving-average in/out filter, by
buy-and-hold, and by *random* trading matched to its own turnover — in every
universe and cost tier tested. The engineering is genuinely good; the strategy has
no edge to capture once measured against the right control.

## Contents

| File | What it is |
|---|---|
| **[Interactive chart ↗](https://druthulu.github.io/algo-trading-reviews/agentic-trading-desk/per-year-chart.html)** | Per-year Return / Max-drawdown / Sharpe for ATD vs the EMA-200 control vs buy-and-hold, across all 4 universes (GitHub Pages) |
| [`initial-thoughts.md`](initial-thoughts.md) | First-read review, before running any backtest (the prior) |
| [`review.md`](review.md) | The full measured findings, why it fails, and what would have to change to earn an edge |
| [`trades-faith11.csv`](trades-faith11.csv) | The full 659-trade ledger on the author's own universe (for anyone who wants to verify a specific trade) |
| `per-year-chart.html` | Source of the interactive chart (self-contained; no dependencies) |

> The chart link is served by **GitHub Pages**. If it 404s, enable Pages once in
> the repo's **Settings → Pages → Deploy from branch → `main` / root**.

## How it was tested (short version)

Split-adjusted daily closes; trailing 290-bar window (the author's spec); signal
on today's close → fill at tomorrow's open; costs at 0 / 3 / 10 bps per side;
dividends credited; densified daily mark-to-market equity curves. The decision
rule and full run matrix were **pre-registered before running**. Controls (same
engine): EMA-200 in/out filter, equal-weight buy-and-hold, and turnover-matched
random trading. Full method is in [`review.md`](review.md).

*Not financial advice — a feasibility audit of an open-source system.*
