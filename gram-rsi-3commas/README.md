# GRAM RSI Strategy [3Commas] — independent review

An independent, pre-registered backtest of the open-source **GRAM RSI Strategy
[3Commas]** (a long-only martingale DCA: buy 4h RSI(14) < 28, average down a fixed
5-rung ladder, take +3% off the blended entry, no stop).

**Verdict: no edge.** The near-100% win rate, tiny drawdown and high Sharpe are
artifacts — of closed-trade accounting, a ~99.9%-idle-cash account, and a benign
window where every dip bounced before the −25% ladder floor could trap it. The
deep-oversold RSI entry has no standalone predictive value; all "profit" is the
averaging mechanic, which is uncompensated for the tail. On US equities over
2016–2026 (COVID, 2022) the same mechanic *did* get trapped.

## Files

- **`review.md`** — the full write-up: what I did, the evidence, why the flattering
  stats are artifacts, the pre-registered test (and why its literal pass is
  overturned), and what would change the verdict.
- **`initial-thoughts.md`** — my prior, written *before* running the backtest.
- **`chart.html`** — interactive, self-contained: mechanic decomposition, the
  cash-deployment illusion, alpha-vs-BTC-beta, drawdown honesty, and the stock
  generalization.
- **`trades-TON.csv`** — the strategy's own trade decisions on TON/USD 4h (the
  system under test's output on public data).

## Method (one paragraph)

The strategy is reimplemented faithfully from its published spec (Wilder RSI(14) on
4h closes, matching TradingView; the 5 safety orders as resting limit orders at the
fixed deviation prices; take-profit recomputed off the blended VWAP after each
fill; no stop). It is driven through a single engine that marks open positions to
market every bar, over offline Kraken 4h data for 13 major coins (2024–2025) and
4h US-equity bars (2016–2026). Harness validity is gated first (no-look-ahead,
cash-conservation invariant, a hand-computed ladder trade, positive controls in
both directions), the decision rule is pre-registered before running, and the
system is split into its two mechanics (entry signal vs averaging) plus
random-turnover-matched and buy-and-hold controls. Not financial advice.
