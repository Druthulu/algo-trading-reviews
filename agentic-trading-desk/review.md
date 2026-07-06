# Agentic Trading Desk — an independent, pre-registered backtest

*Posted to help the author and the community. TL;DR: the engineering is genuinely
good, but as a **strategy** it has no timing edge — it's beaten by a plain
200-day moving-average filter, by buy-and-hold, and by random trading with the
same turnover, in every universe and cost tier I tested. Below is the evidence,
why it fails, and what would have to change to actually earn an edge.*

Not financial advice; this is a feasibility audit of an open-source system.

---

## What I did

I vendored the three ATD scripts **byte-for-byte** (verified by hash) and drove
them over ~9.4 years of daily bars (**2017-02-27 → 2026-06-18**), so the system
under test is exactly the author's code — I only wrote the harness around it.

- **Signals** on split-adjusted closes, trailing 290-bar window (the author's own
  spec). A parity check confirms my in-process calls match running `score.py`
  from the command line on 40/40 sampled days.
- **Execution:** signal on today's close → fill at tomorrow's open. Costs swept at
  0, 3, and 10 bps per side. Dividends credited on ex-date. Curves are densified
  daily mark-to-market (no exit-day-only inflation).
- **Universes (4):** the author's own screenshot holdings (11 names); ~315 liquid
  large/mid-caps; ~500 mid-caps *including delisted names* (so no survivorship
  flattery); ~800 broad. Equal-weight books.
- **Pre-registered** the decision rule and the whole run matrix **before** running
  anything, so nothing here is cherry-picked after the fact.
- **Controls, run through the identical engine:**
  - **EMA-200 in/out** — long when price is above its 200-day EMA, else flat. This
    is the key yardstick: *does all the RSI/MACD/TRIX/pillar machinery beat a
    one-line trend filter?*
  - **Buy & hold** (equal weight).
  - **Random** — random trades matched to ATD's own trade count and hold lengths.
    This isolates *timing skill* from mere exposure and cost.

Sanity checks that make the result trustworthy: buy-and-hold at zero cost matches
an independent total-return calc to the 15th decimal; a cash-conservation
invariant holds to ~1e-15; the EMA-200 control shows its textbook profile (cuts
drawdowns, lags in bull years) — so the harness **can** detect a working timing
rule. It just doesn't detect one here.

---

## What I found: no edge

**Risk-adjusted return (Sharpe), 3 bps, by universe — ATD is last of four, every time:**

| universe | ATD | EMA-200 filter | Buy & hold | Random (matched) |
|---|---|---|---|---|
| author's 11 | **0.35** | 0.61 | 0.70 | 0.58 |
| ~315 liquid | **0.72** | 0.86 | 0.82 | 0.80 |
| ~500 mid | **0.66** | 0.83 | 0.80 | 0.81 |
| ~800 broad | **0.70** | 0.87 | 0.82 | 0.93 |

The most damning line is the last column: **random trading with ATD's own
turnover beats ATD.** Random has zero skill by construction, so being beaten by it
means the entry/exit decisions have *negative* skill — they're worse than no
decision at all.

**Edge vs the control (the pre-registered primary metric).** Paired
block-bootstrap ΔSharpe of ATD minus the EMA-200 filter was **negative in all 12
universe × cost-tier cells** (−0.14 to −0.26). The bar for "edge" was a positive
lower confidence bound; it failed by a wide margin in the wrong direction.

**Absolute alpha.** Factor-neutral (Fama-French 5-factor, Dimson-lagged, HAC)
alpha was **negative** in every universe (−1.7% to −3.9%/yr). The control's alpha
was mildly positive. So ATD isn't neutral-with-no-edge — it's slightly
alpha-*negative*.

**The "downside protection" claim, tested fairly.** ATD *does* draw down less than
buy-and-hold. But (a) the trivial EMA-200 filter delivers the same protection with
a higher Sharpe, and (b) ATD buys that protection by forfeiting most of the
upside. Calendar returns on the author's own book, ATD vs buy-and-hold:

| year | ATD | buy & hold |
|---|---|---|
| 2020 | +20% | **+122%** |
| 2022 | **−19%** | −52% |
| 2023 | +3% | **+67%** |

It only "wins" in the down years (2018, 2022). Its lower drawdown is really just
~49% average market exposure — structural half-in-cash — which you could get more
cheaply by simply holding a smaller position.

**Every variant failed too.** Trim-50%-on-exit, macro-based sizing, tactical
stop/target brackets, same-day-close execution, a 500-bar window, a T+1
settlement lockout, and a shared-cash-pool construction — all pre-registered, none
beat the control. So the easy knobs are exhausted.

**A vivid single case:** TSLA rose ~24× over the window; ATD's *timing* on TSLA
**lost 32%**. The "exit on exhaustion" logic sells strength and rebuys higher.

---

## Why it fails (this dictates the fix)

Two distinct problems:

**1. It fades trends.** "Exit on exhaustion" fires on overbought RSI / a shrinking
MACD histogram — i.e. it **sells strength that tends to continue** — and
"rebound entry" **buys weakness** inside downtrends. That's a mean-reversion bet in
a market that trends. It's exactly why it loses to random: random doesn't have the
anti-momentum bias.

**2. It has no cross-sectional information.** ATD times each ticker in isolation
(a per-name in/out switch). It never ranks names against each other, so it cannot
tell a good name from a bad one — it has no way to *select*, only to *time*.

---

## What it would take to flip "no edge" → "edge"

**Stage 1 — stop fighting the trend (necessary, achievable, but only gets you to a
tie with the filter).** Keep the trend-following skeleton (the EMA-20/50/200
structure, the death-cross exit) and **delete the mean-reversion overlay** — the
RSI/MACD/TRIX "exhaustion" exits and the oversold-rebound entries that are
actively destroying value. Hold winners; exit only on a real trend-break; widen
the bands; decide weekly instead of daily to kill the turnover. The catch: this
converges ATD *toward* the EMA-200 filter, and **that filter itself has no
significant alpha.** You can't beat a price-moving-average rule by tuning a
price-moving-average rule. Stage 1 removes the self-inflicted damage; it doesn't
create an edge.

**Stage 2 — add information the filter doesn't have (sufficient, hard, =
re-architecture).** To beat a price-only trend filter you need a signal it lacks,
in rough order of payoff:

- **Cross-sectional selection (the big one).** Rank the whole universe by relative
  strength / trend quality and hold only the top-K, rebalancing — instead of
  timing each name against itself. This adds *relative* information (which names
  are winning versus each other) that a per-name filter structurally cannot see.
  It's a different program: a ranker, not a flag cascade.
- **A genuine non-price signal** — fundamentals, earnings drift, positioning/flow.
  Daily EMA/RSI/MACD is the most heavily-arbitraged information there is; there's
  essentially no alpha left in the crossovers themselves.
- **Conviction sizing** — size by signal strength, not equal-weight every trigger.
- **Regime conditioning that actually bites** — the macro pillar is currently
  *decorative* (in the author's own code it never changes a trade). If it gated
  exposure, it would reshape the tail — but that buys drawdown, not alpha.

**Honest caveat:** even Stage 2 may not clear a strict bar. Cross-sectional
daily-price momentum is crowded, and rigorous pre-registered testing kills most
plausible-looking signals — including sophisticated fundamental and flow ones. The
same discipline that killed ATD kills a lot of real candidates.

**The blunt version:** for ATD to earn an edge it has to stop being a single-name
technical *timer* and become a cross-sectional *selector*. Stage 1 just removes
the damage; any real edge has to come from Stage 2, and there's no guarantee it
crosses the line.

---

## Credit where due

The engineering is legitimately good: the indicator math is correct (proper
Wilder RSI, SMA-seeded EMAs, correct MACD alignment, population-σ Bollinger), the
"the AI fetches data and the deterministic scripts compute, the human approves"
architecture is the right pattern, and the guardrails are real. None of that is
the problem. The problem is that daily-bar technical timing of single names,
however cleanly implemented, has no edge to capture once you measure it against
the right control.
