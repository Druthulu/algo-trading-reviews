# Agentic Trading Desk — first-read review (before running any backtest)

*My initial reaction after reading the repo — the three scripts, the SKILL/README,
and the screenshots — but before I ran a single backtest. It's a "what's my prior"
note; the measured results (which confirmed the prior) are in **`review.md`**.*

---

## What it actually is

A Claude-skill for **human-in-the-loop swing trading** on stocks/ETFs. The
division of labor is the good part: the AI fetches data and presents analysis,
deterministic Python scripts (stdlib only) do all the math, and the human
approves every order. The core is three files:

- **`indicators.py`** — EMA 20/50/200, Wilder RSI-14, MACD 12/26/9, TRIX-15,
  Bollinger 20/2, computed from a close series.
- **`macro_pillar.py`** — a cross-asset "regime" score from 7 ETF ratios (breadth,
  credit, size, equity-vs-bond, cyclical-vs-defensive) plus an injected 10y-2y
  spread → a −2..+2 pillar.
- **`score.py`** — scores Trend and Momentum −2..+2 each, detects flags
  (exhaustion / relentless-bearish / rebound / death-cross), then a decision
  cascade: enter on rebound, ride, exit on exhaustion, tactical counter-trend
  rebounds at reduced size.

## Initial thoughts

**The engineering posture is genuinely good.** The math is correct — I checked the
implementations: proper Wilder smoothing, SMA-seeded EMAs (TradingView
convention), correct MACD signal alignment, population σ for Bollinger. No
look-ahead in the indicator layer. The "the LLM never estimates math, the scripts
compute, the human decides" architecture is exactly right for what it claims to
be. The rebound detection even has thoughtful touches (an EMA-20 *reclaim*
requires a close below it within the last 5 bars, so a steady uptrend doesn't
constantly fire "rebound"; a TRIX cross only counts on the crossover bar).

**As a strategy, the prior is poor.** It's a bundle of the most heavily-mined
technical-analysis folklore in existence — daily EMA structure, RSI turns, MACD
histogram, death-crosses — with hand-picked thresholds (≥2 exhaustion flags, ≥3
bearish flags, 10% stretch, RSI 45/55) and **zero backtest evidence anywhere in
the repo.** The author never validated it historically; the pitch is about
determinism and guardrails, not performance.

**Three things worth flagging on a first read:**

1. **The macro pillar is decorative.** Despite the "Three-Pillar Framework"
   branding, the decision code only uses the macro score to append warning *text*
   ("reduce size") — it never changes the action, and the −6..+6 composite total
   isn't used in any decision. Mechanically it's two pillars plus flags.
2. **No quantitative risk management.** Stops, targets, and sizing exist only as
   prose ("tight stop," "reduced size"). "EXIT / TRIM" doesn't say how much. A
   mechanical backtest has to invent those numbers.
3. **Structurally, it's a trend filter with churn.** Enter-on-rebound /
   exit-on-exhaustion / stay-out-in-death-cross will approximate "long above the
   200-day, flat below, with extra turnover from the exhaustion exits" — and on a
   settled-cash account the settlement drag compounds the turnover cost the system
   never models.

## Worth backtesting? Yes — cheap, and the answer should be crisp

The signal is a pure function of a close series → a discrete action, single-asset,
long-only, daily bars — which makes it trivial to mechanize and fully
deterministic. My pre-registered prediction before running: **it underperforms
buy-and-hold net of costs in grinding bull markets, and whatever value it has
concentrates in drawdown avoidance during 2022-style regimes** — i.e. it's a
volatility-timing overlay, not an alpha source.

The killer control to run it against is a **plain EMA-200 in/out filter**: if the
flags / TRIX / pillar machinery doesn't beat that one-line baseline, the system is
elaborate decoration on a 200-day filter. (Spoiler, from the full run: it doesn't
— see **`review.md`**.)
