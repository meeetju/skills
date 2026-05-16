---
name: stock-analyser
description: |
  Stock analysis skill combining Elliott Wave Theory (EWT) chart analysis with a
  programmatic tool for data fetching, technical indicators, and charting.

  Use for EWT analysis when the user asks to:
  - Analyse a stock, forex pair, commodity, or any financial instrument using Elliott Waves
  - Find Fibonacci levels for entry, add, or trim decisions
  - Identify which wave we are currently in on a chart
  - Get a position sizing / trade management view based on EWT
  - Perform analysis for tickers formatted as EXCHANGE-SYMBOL (e.g. NYSE-NOW, NASDAQ-AAPL)

  Use for tool-building when working on the stock-analyser project: adding indicators,
  fetching market data, generating charts, or implementing the EWT algorithm in code.

  Trigger phrases: "analyse [ticker]", "where are we in [ticker]", "EWT analysis",
  "Elliott Wave", "fib levels for", "what wave are we in", "add or trim", "wave count",
  "stock analyser", "add indicator", "fetch price data".
---

# Stock Analyser

## Overview

Two modes in one skill:

1. **EWT Chart Analysis** — navigate TradingView, apply Elliott Wave Theory, output a structured trade plan
2. **Tool Building** — programmatic data fetching, indicator computation, charting (language TBD per feature)

See [ewt-algorithm.md](ewt-algorithm.md) for the full wave-counting reference and Fibonacci tables.

---

## Part 1 — EWT Chart Analysis

### Step 0 — Navigate to TradingView

Given a ticker as `EXCHANGE-SYMBOL` (e.g. `NYSE-NOW`, `NASDAQ-AAPL`, `BINANCE-BTCUSDT`):

```
URL = https://pl.tradingview.com/symbols/EXCHANGE-SYMBOL/
```

**Timeframe sequence:** Weekly → Daily → 4H/1H (high-degree first, precise entry last).

### Step 1 — Identify Primary Trend (High-Low Method)

Find the most recent significant swing high (downtrend) or swing low (uptrend) as the wave count start.

- **Sharp, non-overlapping, strong momentum** → Motive Phase (waves 1–5)
- **Choppy, overlapping** → Corrective Phase (A-B-C or complex)

Confirm with Higher Highs/Higher Lows (uptrend) or Lower Highs/Lower Lows (downtrend).

### Step 2 — Wave Count

Apply the Elliott Wave framework from [ewt-algorithm.md](ewt-algorithm.md). Key hard rules (never violated):

| Rule | Condition |
|------|-----------|
| Rule 1 | Wave 2 never retraces past the start of Wave 1 |
| Rule 2 | Wave 3 is never the shortest of Waves 1, 3, 5 |
| Rule 3 | Wave 4 never enters Wave 2 territory |

### Step 3 — Fibonacci Levels

| Wave | Draw From → To | Zone |
|------|----------------|------|
| Wave 2 end | W1 start → W1 end | 50%–61.8% retrace |
| Wave 3 end | W1 start → W1 end (expand from W2) | 161.8%–200% extension |
| Wave 4 end | W3 start → W3 end | 23.6%–38.2% retrace |
| Wave 5 end | W4 end → W4 inverse length | 100%–161.8% extension |
| ZZ Wave B | A start → A end | 50%–61.8% retrace |
| ZZ Wave C | A start → A end (from B) | 100% extension |

Full corrective and complex correction tables: see [ewt-algorithm.md](ewt-algorithm.md).

### Step 4 — RSI Divergence Validation

RSI(14). Divergence alone is not a signal — combine with valid wave count.

- **Bullish divergence** (price LL, RSI HL) → confirms end of Wave 5 down or Wave C down
- **Bearish divergence** (price HH, RSI LH) → confirms end of Wave 5 up or Wave C up

Most powerful between Wave 3 and Wave 5 peaks/troughs.

### Step 5 — Chart Pattern Cross-Check

| Pattern | EWT Location | Implication |
|---------|-------------|-------------|
| Rising/Falling Wedge | Ending Diagonal in W5 or C | Sharp reversal |
| Contracting Triangle | Wave 4 or Wave B | Thrust coming |
| Head & Shoulders | End of Wave 5 up | Wave A down starting |
| Inverse H&S | End of Wave 5 down | Wave A up starting |
| Double Top/Bottom | End of Wave 5 | Flat correction likely |

### Step 6 — Analysis Output

```
═══════════════════════════════════════════════════
EWT ANALYSIS: [TICKER] | [DATE] | [TIMEFRAME]
═══════════════════════════════════════════════════

📍 CURRENT PRICE: [price]

🌊 WAVE POSITION
   Primary trend:    [Bullish / Bearish]
   Phase:            [Motive / Corrective]
   Current wave:     [e.g. "Wave 4 of Impulse up from [date/price]"]
   Sub-wave:         [e.g. "Wave (b) of Flat correction"]
   Degree:           [Primary / Intermediate / Minor]
   Confidence:       [High / Medium / Low] — [brief reason]

📐 FIBONACCI LEVELS
   Wave count drawn from: [start price] → [end price]

   🟢 ADD / BUY ZONES:
      Level 1: [price] (38.2%)
      Level 2: [price] (50.0%)  ← Primary
      Level 3: [price] (61.8%)  ← Deep pullback

   🎯 TRIM / SELL ZONES:
      Target 1: [price] (100% extension)
      Target 2: [price] (161.8%)  ← Primary
      Target 3: [price] (200%)    ← Extended

   🔴 STOP LOSS:
      Hard stop: [price] ([wave rule violated])

⚠️  INVALIDATION
   [Price/movement that invalidates the count]

📊 RSI DIVERGENCE
   [Present / Absent] — [description]

🔲 CHART PATTERN
   [Pattern] → [Wave match]

💡 TRADE MANAGEMENT SUMMARY
   Current situation: [1-2 sentences]

   If LONG:
   - Add at: [level] and [level]
   - Trim 1/3 at: [target 1]
   - Trim 1/3 at: [target 2]
   - Trail stop to break-even after Target 1 hit
   - Hard stop: [stop level]

   If SHORT:
   - Add at: [level] and [level]
   - Cover 1/3 at: [target 1]
   - Cover 1/3 at: [target 2]
   - Hard stop: [stop level]

📝 ALTERNATIVE COUNT
   [If confidence < High: describe alternative interpretation]
═══════════════════════════════════════════════════
```

### Quick Reference — Wave Personality

| Wave | Character | RSI |
|------|-----------|-----|
| 1 | Weak, doubted | Recovering from oversold |
| 2 | Deep pullback — looks like trend resuming | Pulls back, stays above 30 |
| 3 | **Strongest, longest, most volume** | Hits overbought (>70) |
| 4 | Choppy, frustrating sideways | 40–60 range |
| 5 | Narrowing breadth, divergence begins | Diverges vs Wave 3 |
| A | Sharp first correction leg | Breaks below 50 |
| B | Counter-trend rally — bull/bear trap | Bounces, fails |
| C | Final correction leg — strongest corrective | Reaches oversold |

**Most tradeable:** Wave 3 > Wave C (ZZ) > Wave 5 (with divergence) > Wave A  
**Avoid:** Wave 2, Wave B, inside a Triangle

---

## Part 2 — Tool Building

Language and framework are **not fixed** — choose the best fit per feature and document it.

### Data fetching

Sources (preference order):
1. **yfinance** (Python) — zero-cost, OHLCV history, no key needed
2. **Alpha Vantage** — free tier, API key required
3. **Polygon.io** — reliable paid option

Fields: OHLCV + adjusted close + ticker metadata (currency, exchange).

Conventions:
- Store raw fetched data before computing anything
- Timestamps in UTC; convert to local only for display
- Cache locally: `data/<ticker>_<date>.parquet` or `.csv`

### Technical indicators

Implement as pure functions: `fn(prices) → series`. Return NaN (not zero) for insufficient data.

| Indicator | Inputs | Notes |
|-----------|--------|-------|
| SMA(n) | close, window n | |
| EMA(n) | close, window n | |
| RSI(14) | close | Wilder smoothing; clamp 0–100 |
| MACD | close | EMA(12)−EMA(26); signal = EMA(9) of MACD |
| Bollinger Bands | close, 20, σ=2 | Upper/lower = SMA ± 2σ |
| ATR(14) | high, low, close | |

Python: use **pandas-ta** or **ta-lib**; hand-roll only when unavailable.

### Charting

- Candlestick + volume sub-panel — primary view
- Indicator overlays on price panel (SMA/EMA/Bollinger)
- Oscillator sub-panel (RSI, MACD)

Library: Python → **plotly** (interactive HTML) or **matplotlib** (static PNG).  
Output: `charts/<ticker>_<indicator>_<date>.html` — always include legend and axis labels.

### EWT algorithm (future Rust implementation)

The EWT analysis in Part 1 is currently manual (TradingView + screenshots). The end goal is a Rust implementation. See [ewt-algorithm.md](ewt-algorithm.md) for the pseudocode and data structures.

### Project layout

```
stock-analyser/
  data/       # cached raw OHLCV files
  charts/     # generated chart outputs
  src/        # source code
  tests/      # unit tests for indicators
  README.md
```

---

## When learning new things for this project

1. Use the **note-taker** skill to append findings to `~/GitRepos/book/` (e.g. `python.md`, `rust.md`).
2. Update this SKILL.md if the finding changes how the project should be built.
3. Update [ewt-algorithm.md](ewt-algorithm.md) if the finding refines the wave-counting or Fibonacci methodology.
