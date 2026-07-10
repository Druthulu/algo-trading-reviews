# GRAM RSI Strategy [3Commas] — first-read review (before running the backtest)

*My initial reaction after reading the published strategy (the reddit post, the
TradingView description, the reported Strategy Report) — but before I ran a single
independent backtest. It's a "what's my prior" note; the measured results are in
**`review.md`**.*

Not financial advice; this is a feasibility audit of an open-source strategy.

---

## What it actually is

A **long-only martingale DCA** on GRAM/USDT, 4-hour bars. One position at a time:

- **Entry (base order):** a long opens when 4h RSI(14) prints **below 28** (deep
  oversold) and there is no open position. Sampled at bar close, no repaint.
- **Averaging ladder (5 safety orders):** at **fixed** deviations from the base
  entry — **−2 / −5 / −9.5 / −16 / −25%** — each larger than the last, sizes
  scaling **1.8×** (base 500 → 900 / 1,620 / 2,916 / 5,249 / 9,448 USDT). Deeper
  rungs pull the blended average entry down toward the latest fill.
- **Exit:** a fixed **3% take-profit above the running average entry**. No
  trailing.
- **Risk:** **no stop-loss.** Max deployed is base + 5 AOs ≈ 20,633 USDT (~20.6%
  of a 100k account); the −25% ladder floor is the only structural cap.

Reported backtest (GRAM/USDT 4h, Jan 2024–Jul 2026): **+3.82% net, 2.21% max
drawdown, 78 trades, 69.23% win rate, profit factor 4.79.**

One thing up front: **the author is unusually candid.** The write-up itself flags
the sub-100-trade sample, calls the system a martingale, notes that "part of that
PF is the averaging mechanic itself — not a directional edge," and warns that the
2.21% drawdown is closed-trade drawdown over a window where dips recovered. That
honesty is rare and worth crediting. This review sets out to *quantify* exactly
the caveats the author already gestures at.

**A crucial identity for reading the numbers:** "GRAM" is **Toncoin (TON)
rebranded** (the ticker changed from TON to GRAM in mid-2026). So the ~30-month
"GRAM/USDT" history is really *Toncoin* price history — which matters, because
**TON fell about 80% peak-to-trough inside the test window.** Hold that thought.

## Engineering

**The spec is clean and the engineering is transparent.** RSI(14) on 4h closes is
a Wilder RMA (TradingView's `ta.rsi`), reproducible to the decimal; the ladder is
a fully-specified, fixed schedule (not an opaque adaptive grid); the entry is
evaluated on closed bars, so there's no look-ahead in the signal. The webhook
payloads to drive a DCA bot are a nice touch. Nothing here is sloppy — it does
exactly what it says.

## As a strategy, the prior is poor — for three specific, measurable reasons

**1. The headline "2.21% max drawdown" is the wrong drawdown.** It is *closed-trade*
equity drawdown — it only moves when a trade closes. But a martingale's risk lives
in the **open** position: after all five safety orders fill, you are holding a
full ~20.6%-of-equity bag with **no stop**, and if price keeps falling you sit
there, marked underwater, until it either recovers 3% off your averaged entry or
you give up. The honest number is the **mark-to-market drawdown including the open
position** — and on an asset that fell ~80% in-sample, I expect that to dwarf
2.21%.

**2. The win rate and profit factor are mechanical, not predictive.** A fixed 3%
take-profit off an *averaged-down* entry converts most oversold dips into small,
quick wins — of course the win rate is ~70%. The losses are the rare dips that
*don't* bounce, and with no stop they are open-ended. That is textbook **negative
skew**: many small wins, few (uncapped) large losses. PF 4.79 and WR 69% are what
that distribution *looks like* over a lucky window; they are not evidence of a
timing edge.

**3. +3.82% over ~30 months is barely a pulse — and it's on ≤20.6% deployed.** Most
of the capital sits idle (RSI<28 is infrequent), so the tiny total return and the
tiny drawdown are two sides of the same coin: a small bet. The fair questions are
(a) what did it earn *on the capital actually at risk*, and (b) did any of it beat
simply **holding the asset** — or **holding BTC** — over the same window?

## The one question that matters for "is any of this real": which mechanic pays?

The system bundles two things: a **timing signal** (buy deep-oversold RSI) and a
**position-construction trick** (a martingale averaging ladder with a fixed small
TP). To know whether there's anything transferable here, you have to separate them.
So I'll run a 2×2 — {RSI gate on/off} × {ladder on/off} — with the **shipped
defaults unchanged**:

- **S_full** = RSI<28 + ladder (the shipped strategy)
- **S_gate_only** = RSI<28 + a single base order (does the *entry signal alone* pay?)
- **S_ladder_nogate** = no RSI gate + ladder (does the *averaging alone* pay?)
- **S_random_ladder** = random entries, turnover-matched + ladder (timing skill vs none)

plus **buy-and-hold** the asset and **buy-and-hold BTC** as yardsticks — all through
the identical engine, marked to market.

## Pre-registered prediction (before I look)

1. **Drawdown:** the mark-to-market max drawdown (open positions included) is
   **materially larger** than the reported 2.21% — badly so on TON specifically.
2. **The RSI gate is close to decoration:** S_full does **not** beat
   S_ladder_nogate by a significant risk-adjusted margin — the averaging mechanic,
   not the oversold timing, does most of the work. And S_full does **not** beat
   random-turnover-matched entries with the same ladder.
3. **No edge over holding:** on a risk-adjusted (Sharpe) basis, neither S_full nor
   its variants beats buy-and-hold of the asset, pooled across the majors.
4. **Generalization:** the mechanic "works" on other majors and on stocks only in
   the sense that the **same illusion** reappears — high win rate, small gains, a
   rare deep tail — not a genuine, portable edge.

**The killer control** is S_full vs **S_ladder_nogate**: if the deep-oversold RSI
gate doesn't beat a no-gate averaging ladder, then the strategy's "edge" is just
the averaging mechanic betting that dips recover — a bet that is uncompensated for
the one dip that doesn't. (Spoiler, from the full run: see **`review.md`**.)
