# GRAM RSI Strategy [3Commas] — an independent, pre-registered backtest

*Posted to help the author and the community. TL;DR: the strategy is honest about
being a martingale, and it does exactly what it says — but its flattering numbers
(near-100% win rate, tiny drawdown, high Sharpe) are **artifacts**, not edge. The
deep-oversold RSI entry has **no standalone predictive value**; every dollar of
"profit" comes from the averaging mechanic, which is uncompensated for the tail;
and the account is **~99.9% idle cash**, so the low drawdown and high Sharpe are a
near-zero-deployment illusion. It fired zero losing trades in-sample only because
the 2024–25 window let every dip bounce before the −25% ladder floor could trap
it — on equities over 2016–2026 (COVID, 2022), the same mechanic **did** get
trapped. Evidence below, with what would change the verdict.*

Not financial advice; this is a feasibility audit of an open-source strategy.

---

## What the strategy is (one line)

Long-only DCA: open a base order when 4h RSI(14) < 28; average down on a fixed
5-rung safety-order ladder (−2/−5/−9.5/−16/−25%, sizes ×1.8 from a 900 USDT
first rung on a 500 base); close the whole position at **+3% above the blended
average entry**; **no stop-loss.** Reported backtest: +3.82% net, 2.21% max
drawdown, 78 trades, 69% win rate, profit factor 4.79.

**One identity that matters:** "GRAM" is **Toncoin (TON) rebranded** — the 4h
history is Toncoin, which fell **~80% peak-to-trough** inside the test window.

## What I did

I reimplemented the strategy **faithfully** from its published spec (Wilder RSI(14)
on 4h closes, exactly matching TradingView; the 5-rung ladder as resting limit
orders at the fixed deviation prices; TP recomputed off the blended VWAP after
each fill; no stop) and drove it over **offline Kraken 4h bars** for **13 major
coins** (BTC, ETH, SOL, XRP, ADA, LINK, AVAX, DOT, LTC, ATOM, POL, JUP, and TON),
**Jan 2024 – Dec 2025**. Because Kraken lists TON only from Oct 2024 and my archive
ends Dec 2025, this is an **independent test on a different venue and a shorter
window — not a reproduction** of the author's exact 78-trade number.

To find out *whether any of it is real*, I split the strategy into its two
mechanics — an entry **signal** and a position-construction **trick** — and ran a
2×2 with the **shipped defaults unchanged** on every asset:

| arm | what it is | what it isolates |
|---|---|---|
| **S_full** | RSI<28 + 5-rung ladder | the shipped strategy |
| **S_gate_only** | RSI<28 + a single base order | does the *entry signal alone* pay? |
| **S_ladder_nogate** | no RSI gate + ladder | does the *averaging alone* pay? |
| **S_random_ladder** | random entries, turnover-matched + ladder | timing skill vs none |

plus buy-and-hold of the asset and of BTC. Everything runs through one engine that
marks **open positions to market every bar** (so drawdown includes underwater
ladders), pre-registered before running. Harness validity was gated first: a
no-look-ahead property test, a cash-conservation invariant (< 1e-9), a
hand-computed ladder trade the engine must reproduce exactly, and positive
controls in both directions (a recovering series must profit; a sustained decline
must trap the ladder). All green.

---

## What I found

### 1. The RSI entry has no edge; the averaging does all the work

Pooled mean across the 13 majors, 6 bps/side:

| arm | mean net % | mean daily Sharpe | reading |
|---|---:|---:|---|
| **S_full** (RSI + ladder) | **+3.76** | 1.93 | the shipped strategy |
| **S_gate_only** (entry signal alone) | **+0.10** | 0.25 | **the RSI entry, stripped of averaging, makes nothing** |
| **S_ladder_nogate** (averaging alone) | **+14.88** | 1.04 | the averaging mechanic is the entire engine |
| buy & hold the asset | −12.23 (median −44.0) | 0.10 | yardstick |

*Net % = account return over the window. Daily Sharpe = annualized mean/σ of
daily account returns.* The single most important line is **S_gate_only ≈ +0.10%**:
if the deep-oversold RSI entry had genuine predictive value, buying it with one
order and taking +3% would make money. It doesn't. The "profit" lives entirely in
the averaging — which is not a forecast, it's a bet that price mean-reverts +3%
before it falls another 25%.

### 2. The high Sharpe and tiny drawdown are a ~99.9%-cash illusion

| deployment metric | value |
|---|---:|
| **average capital actually deployed** | **0.13% of the account** |
| average *max* deployed (all AOs) | 8.5% |
| average time in the market | 5.8% of bars |

The strategy is in cash almost always. An account that is 99.9% idle has almost no
volatility, so its Sharpe looks high and its drawdown looks tiny — but that says
nothing about the quality of the trades. This is also why it "beats" buy-and-hold
on the alts that crashed (mean −12%, median −44%): **it wasn't in the market.**
And on the assets that rose — BTC +107%, XRP +199%, ETH +30% — it captured almost
none of it (+1.6%, +4.5%, +5.0%).

### 3. The reported "2.21% max drawdown" is the wrong drawdown — and even the right one is small *only because the tail never fired*

| drawdown measure | value | what it means |
|---|---:|---|
| author reported (closed-trade) | 2.21% | only moves when a trade *closes* |
| my closed-trade DD (worst asset) | 0.00% | in-sample, **no trade ever closed at a loss** |
| account MTM DD, open positions included (worst asset) | 1.14% | tiny — because deployment is tiny |
| worst position underwater (worst asset) | 10.1% | how deep a live ladder got, on its own capital |
| times fully loaded (all 5 AOs) | 13 | it *did* reach the ladder floor... |
| forced losing closes in-sample | **0** | ...but always escaped on a bounce |
| synthetic monotonic −56% crash → | **−45.8% trade, 9.5% account DD, force-closed** | the regime that didn't occur |

Across 13 coins over two years — several of which fell 70–90% — **not one position
closed at a loss.** The ladder fully loaded 13 times and still escaped every time,
because the declines were *choppy* enough to hand back a 3% bounce off the
averaged-down entry. That is the author's own caveat, measured: the tiny drawdown
is **survivorship over a lucky window**, not robustness. Feed the same ladder a
*smooth* −56% decline with no bounce and it loses 46% on the trade and force-closes
— the latent tail the no-stop design carries.

### 4. It generalizes to stocks — as the same illusion, and there the tail *does* fire

The identical mechanic on 4h US-equity bars over **2016–2026** (including the 2020
COVID crash and the 2022 bear):

| ticker | S_full net % (decade) | S_full Sharpe | buy & hold | no-gate ladder: forced loss? worst underwater |
|---|---:|---:|---:|---|
| SPY | +2.78 | 0.20 | **+266%** | yes · 20% |
| QQQ | +3.29 | 0.35 | **+561%** | yes · 15% |
| AAPL | +4.02 | 0.44 | **+1,029%** | yes · 18% |
| NVDA | +5.89 | 0.31 | **+25,807%** | yes · 51% |
| TSLA | +9.01 | 0.67 | **+2,397%** | yes · 59% |

Over a **decade**, S_full earned low-single-digits while the assets did hundreds to
tens-of-thousands of percent. And the always-in averaging ladder took a **forced
loss on every name** and went **31–58% underwater** through 2020/2022 — the tail
that stayed hidden in the benign 2024–25 crypto window is real; it just needs a
sustained, non-choppy decline to bite.

---

## The pre-registered test — and why I overturn its literal verdict

My pre-registered primary metric was the paired block-bootstrap ΔSharpe of
**S_full − S_ladder_nogate** (does the RSI gate add risk-adjusted value over a
no-gate ladder), with a GO if its lower bound and the ΔSharpe-vs-buy&hold lower
bound both cleared zero. Here is the paired battery (pooled, 6 bps):

| comparison | ΔSharpe | 90% CI | p(better) |
|---|---:|---|---:|
| S_full − S_ladder_nogate | +1.40 | [+0.17, +2.84] | 0.98 |
| S_full − buy & hold | +2.51 | [+1.20, +4.04] | 1.00 |
| S_full − random-timing (matched) | **+0.63** | **[−0.16, +1.65]** | 0.95 |
| S_full − deployment-matched buy&hold | +2.77 | [+1.45, +4.30] | 1.00 |

By the **literal** rule this is a GO — S_full's Sharpe clears both bars. **I
overturn it**, and here is exactly why, because the metric is confounded:

1. **The disambiguating control fails.** Against **random entries at the same
   turnover**, the ΔSharpe CI **includes zero** (+0.63, [−0.16, +1.65]). The RSI
   *timing* is statistically indistinguishable from random. S_full only "beats"
   the no-gate ladder and buy-and-hold on Sharpe because it **holds even more
   cash** — an exposure artifact, not skill.
2. **The signal makes nothing on its own** (S_gate_only ≈ +0.10%). A real entry
   edge would survive without the averaging crutch.
3. **The gate has *less* market-neutral alpha than the no-gate ladder.** Regressed
   on BTC (Newey-West):

   | arm | annual alpha | BTC beta | t(alpha) |
   |---|---:|---:|---:|
   | S_full | +1.28% | ≈0.00 | +4.33 |
   | S_ladder_nogate | +2.67% | 0.07 | +1.90 |
   | buy & hold | −26.4% | 1.14 | −1.50 |

   S_full's +1.28%/yr is **real in-sample** (market-neutral, t=4.33) — but it is
   *smaller* than the no-gate ladder's alpha, so the RSI gate **subtracts** alpha
   while adding Sharpe-by-sitting-out. That +1.28% is the mean-reversion-harvest
   premium — the fee for writing naked crash-insurance, collected in a window with
   no crash — not a timing edge. One tail event (§3) erases many years of it.

So: **the shipped statistics pass a naive bar, but every test designed to
separate skill from artifact says there is no edge in the RSI entry.** Verdict:
**NO EDGE.** The strategy is a cash-parked mean-reversion carry whose safety is
borrowed from a benign regime.

## What would flip this to "edge"

- **S_gate_only turns clearly profitable** across assets and cost tiers — i.e. the
  RSI<28 entry, with *no* averaging, has standalone forward-return predictive
  value. (It doesn't: ≈0% on crypto, ≈0.5%/decade on stocks.)
- **S_full beats random-timing** with a paired ΔSharpe lower bound **above zero**
  (it doesn't: [−0.16, +1.65]).
- The edge survives on **return-on-capital-deployed** terms rather than on a
  99.9%-idle account, *and* survives a realistic tail (a sustained non-choppy
  decline) rather than depending on the ladder never being trapped.

None of these hold. The pieces that would make it real are exactly the pieces that
are absent.

## Credit where due

The author is refreshingly honest — the write-up itself flags the sub-100-trade
sample, calls the system a martingale, notes that "part of the profit factor is the
averaging mechanic, not a directional edge," and warns the 2.21% drawdown depended
on dips recovering. All true. The engineering is clean: correct Wilder RSI, a
transparent fixed ladder, no look-ahead in the signal, tidy webhook plumbing. The
problem isn't the code or the candor — it's that a deep-oversold RSI entry has no
edge to capture once you separate it from the averaging, and the averaging is a
short-tail carry that a good backtest window flatters and a bad one destroys.
