# algo-trading-reviews

Independent, pre-registered backtests of open-source algorithmic-trading systems —
run against real controls (a trivial baseline, buy-and-hold, and turnover-matched
random) so "does it actually have an edge?" gets a straight, reproducible answer.

Each review vendors the system under test unchanged, drives it over multi-year
daily data with realistic fills/costs/dividends, pre-registers the decision rule
*before* running, and reports the result with bootstrap confidence intervals and a
gross-vs-net (0-bps) decomposition.

## Reviews

| System | Verdict | Folder |
|---|---|---|
| [Agentic Trading Desk](https://github.com/Oft3r/agentic-trading-desk) (Oft3r) | No timing edge | [`agentic-trading-desk/`](agentic-trading-desk/) |

## Interactive charts

Charts are served via **GitHub Pages**. To enable (once): repo **Settings → Pages
→ Deploy from a branch → `main` / root**. Links live in each review's README.

*Not financial advice. Each review is a feasibility audit, not a recommendation.*
