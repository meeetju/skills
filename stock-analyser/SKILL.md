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

**Default horizon:** start with at least the last **3 years** on Weekly/Daily first, then refine on lower timeframes.

**Chart framing rule (required):**
- On higher-timeframe charts, keep the **current price near the middle of the visible range** at minimum (not pinned to the right edge) so prior structure is not lost.
- Pan/zoom until at least one full prior major cycle is visible before assigning wave labels.

**Anchor priority for levels/counts:**
1. User-provided chart anchors/labels (highest priority)
2. Direct chart structure (swing highs/lows, MA, Fib)
3. External quote/news data (context only, not primary for wave anchors)

**Self-sufficient analysis rule (required):**
- Never block analysis waiting for the user to upload a chart.
- First produce the best analysis available from accessible chart/market data.
- If chart fidelity is limited, clearly mark confidence and assumptions, then optionally ask for anchors to refine.

**Response style default (concise, top-loaded):** begin every analysis with a compact snapshot:
- Current price
- Current wave and phase
- Wave start / key anchor prices used
- Next likely targets (with Fib labels)
- Invalidation / bear-case trigger

**Investing cycle order (apply in sequence):**
- Confirm fundamentals remain undervalued first
- Classify trend phase (impulse vs corrective) and current wave
- Prefer early build near Wave 2 completion when valid
- Set and manage Wave 3/Wave 5 targets, trim and re-add through Wave 4 support
- Recycle through ABC (re-enter after A, trim B rejection, reassess at C)
- Keep re-validating fundamentals and risk sizing throughout

### Step 1 — Identify Primary Trend (High-Low Method)

Find the most recent significant swing high (downtrend) or swing low (uptrend) as the wave count start.

- **Sharp, non-overlapping, strong momentum** → Motive Phase (waves 1–5)
- **Choppy, overlapping** → Corrective Phase (A-B-C or complex)

Confirm with Higher Highs/Higher Lows (uptrend) or Lower Highs/Lower Lows (downtrend).

**Down-trend confirmation checklist** (use confluence; do not rely on one signal):
- **Structure:** sequence of Lower Highs + Lower Lows
- **Moving averages:** price stays below key MA (e.g. 50/200), or bearish MA crossover
- **Support/resistance:** prior support breaks and then acts as resistance on retest
- **Fib pullbacks:** counter-trend bounces reject near common retrace zones before new lows
- **Patterns:** bearish channel continuation or bear-flag breakdown
- **Volume/momentum:** stronger volume on sell legs than on relief rallies
- **EWT context:** impulsive legs tend to point down (1-3-5) with corrective rebounds in between

**Support confirmation checklist** (bullish case, use confluence):
- Draw horizontal support/resistance zones from prior lows (blue line convention)
- Prefer overlap of structure + MA + Fib (W2: 0.50–0.618, W4: ~0.382, Wave C: 0.50–0.618)
- Accept either: single V-bounce or multi-test base with HH/HL progression
- Strengthen confidence with bounce volume, bullish candles, and RSI/MACD divergence
- Downgrade confidence ahead of binary events (earnings/guidance, investigations, macro shock risk)

**BUY ZONE construction (multi-anchor confluence — see `graphs/onds-wave3-vs-wave1-case.png` and `graphs/crm-buy-zone-band-case.png`):**
- A BUY ZONE is **always a band**, not a single Fib level. The band has explicit **upper edge** and **lower edge** Fibs.
- **Default Fib bands by wave identity:**
  - Shallow Wave 4 (in-progress impulse): `0.236 → 0.50` of bigger Wave 3
  - Deep correction (post-impulse, bigger Wave 2 / `(C)` of higher-degree A-B-C): **`0.786 → 0.886`** of the entire prior impulse — *this is the most common BUY ZONE band on weekly charts*
  - Standard Wave 2 of a still-fresh impulse: `0.50 → 0.618`
- Build the band from the overlap of *at least two* of the following anchors:
  - Retracement of the **bigger prior wave** (Fib upper + lower edge)
  - Retracement of the **immediately prior sub-leg** (e.g. `0.50–0.786` of sub-`(B)`)
  - Sub-wave-`(C)` typical zone (`0.5–0.618` of sub-`(A)` projected from sub-`(B)` peak, or `(C) ≈ 1.0–1.618 × (A)`)
  - Prior **structural support** (horizontal blue line)
  - **Trendline** support (yellow ascending or descending)
  - Key **moving average** (50/200 SMA)
- A **single Fib retrace by itself is not a BUY ZONE**. Cite all anchors in the analysis output.

**Tiered-entry playbook (mandatory when price is at or inside the BUY ZONE band):**

| Tier | Trigger | Recommendation |
|------|---------|---------------|
| **Starter** | Price *inside* the BUY ZONE band (between upper and lower Fib edges), **or** at a single MA + Fib confluence (see below) | **Starter BUY** is acceptable now (understandable, partial size). Don't wait for the band low — being inside the band IS the trigger. |
| **Lower-risk add — MA flip** | Daily close *above* the closest MA above price (typically 50 DMA) **and** a successful retest hold of that MA as support | Add on the retest hold. |
| **Lower-risk add — structural flip** | Daily close *above* the prior **swing-high horizontal line** (the chart-marked blue line) **and** a successful retest hold (resistance flips to support) | Add on the retest hold. Often comes *before* the next MA flip. |
| **Trend confirmation add** | Daily close *above* the next higher MA (typically 200 DMA) | Add on confirmation; ride toward post-correction targets. |

**MA + Fib confluence is itself a BUY ZONE (do not wait for a deeper Fib):**
- When the **50 DMA** aligns with a **`0.618`** retrace of the immediately prior sub-leg (or any other Fib level), that single price counts as a confluence BUY ZONE — *equivalent* to a multi-Fib band.
- Same applies to 200 DMA + `0.786` / `0.886` confluences for deeper buys.
- **Always print the current 50 DMA and 200 DMA prices** in the analysis, and tag them as triggers/targets. Use them as the *primary* anchor when they overlap a Fib.

**Treat key MAs and prior swing-high lines as both triggers and targets:**
- 50 DMA reclaim (with retest hold) = **lower-risk add trigger** (MA flip)
- Prior swing-high horizontal line reclaim (with retest hold) = **lower-risk add trigger** (structural flip) — often the *first* signal in time, before the MA flip
- 200 DMA = **first upside target after the flips**
- Always print these as named, numbered levels in the QUICK MAP and the BULL CASE block.

**Sub-degree awareness (critical for entries inside an active impulse):**
- While bigger Wave `3` is running, every higher-impulse sub-leg has its own internal `(1)(2)(3)(4)(5)`. *Each* sub-`(2)` is a valid trade — sometimes you are pulled-back inside **sub-`(2)` of sub-`(3)`**, not sub-`(2)` of bigger `3`.
- **Identify which degree the current pullback belongs to before drawing Fibs.** Anchor sub-`(2)` Fibs on the *most recent* completed sub-`(1)` of the *current* impulse degree, not on the much larger leg from the bigger Wave `2` low.
- The smaller-degree sub-`(2)` typically retraces only `0.382 → 0.618` of its own sub-`(1)` (shallower than a bigger-degree Wave 2). A `0.618` here, especially with MA confluence, is the right buy level — even if it sits well above your bigger-leg projections.

**Recommendation rule when price is inside the BUY ZONE band:**
- Default to **BUY (starter)** — never `HOLD` or `WAIT` simply because price hasn't touched the band's lower Fib. Waiting for the deep edge is *optional* (deeper-add tier), not the gate for any participation.
- If you want to be cautious, downgrade size (smaller starter), not the recommendation itself.

**Big-bounce signal (do not dismiss):** a daily `>15%` green bar on multi-multiple-of-average volume off a confluence BUY ZONE is high-probability sub-`(C)` completion. Read this as a *structural* signal, not a knife-catch trap, especially when the prior count was a corrective `(A)–(B)–(C)`.

**Moving-average baseline (SMA 50/200):**
- Prioritize **Daily** and **1H** for this check
- Above SMA 200 usually favors bulls; below SMA 200 usually favors bears
- SMA 50 crossing above SMA 200 = golden cross (bullish bias), below = bearish bias
- Use MA bias with structure + wave count + Fib; never as a standalone signal
- Keep chart setup consistent across platforms (TradingView/Webull): SMA 50 in blue, SMA 200 in yellow

### Step 2 — Wave Count

Apply [ewt-algorithm.md](ewt-algorithm.md) for impulse detail. Four types: default **normal**; use **Extended** when one motive leg (often W3) is dramatically longer — especially after major news/volatility; use **Diagonal** when overlap rules apply (see reference).

For primary outputs, anchor the count on the 3-year structure first (Weekly/Daily), with current price centered in view, then map current sub-wave.

**Wave-degree notation (always state both degrees):**
- **Higher / bigger trend** (primary count): bare numerals **without parentheses** — `1 2 3 4 5` for impulse, `A B C` for correction, or Roman `I II III IV V` / `A B C` if a degree above is also referenced.
- **Lower / shorter-term sub-count** (one degree below): Arabic numerals **with parentheses** — `(1) (2) (3) (4) (5)` and `(A) (B) (C)`.
- **Sub-sub count** (two degrees below) when needed: circled `①②③④⑤` / `Ⓐ Ⓑ Ⓒ` — used when the chart already shows a `(1)–(5)` count and a finer count must coexist.
- Always make explicit which sub-wave belongs to which higher wave, e.g. *“sub-wave `(1)` of bigger Wave `III`”*, *“sub-wave `(C)` of bigger Wave `II`”*.
- See `graphs/oscr-wave-degrees-case.png` (two-degree case: bigger `II` → bigger `III` with sub-`(1)/(2)`) and `graphs/onds-wave3-vs-wave1-case.png` (three-degree case: bigger `2` → bigger `3` shown via circled `③④⑤` → bigger `4` shown as `(A)(B)(C)`).

**Wave-1 vs Wave-3 identification (critical — covered in `graphs/onds-wave3-vs-wave1-case.png`):**
- A **violently parabolic move out of a multi-year base** (e.g. `>10×` in `<2 years`, vertical advance with extension) is almost always a **Wave 3** of a larger impulse, **not Wave 1**.
- The prior multi-year base/consolidation is the bigger **Wave 1 + Wave 2**, with the breakout point being the bigger Wave 2 terminal low.
- **Consequence for the next correction:** it is bigger **Wave 4 (shallow)**, **not bigger Wave 2 (deep)**:
  - bigger Wave 4 retraces typically `0.236–0.50` of bigger Wave 3 (and must respect Rule 3 — never into bigger Wave 2 territory)
  - the corrective structure is most often `(A)–(B)–(C)` with `(C)` landing in the buy zone
- **Consequence for sizing:** target the next impulse as bigger **Wave 5** (`1.0`–`1.618` of bigger `1`, or `1.0`–`1.618` projected from bigger Wave 4 low using bigger Wave 1 length), not as bigger Wave 3 of a fresh impulse. Don't anchor `2.618` projections off this wrong identification.
- **Heuristics that betray a Wave 3 (not a Wave 1):**
  - Move is `≥5–10×` in under two years
  - Multi-year base before it (sideways or extreme low)
  - Mass-media attention / parabolic volume in the latter third
  - RSI hits extreme overbought multiple times during the rally
- When in doubt, **default the recent parabolic rally to bigger Wave 3** unless the structural evidence clearly supports Wave 1.

**Complete-impulse vs in-progress-impulse — the most common count error (see `graphs/meta-complete-impulse-case.png`):**

Before anchoring any bigger-wave count, check whether the rally on the chart is **still unfolding** as a single Wave 3 or whether it is a **complete `1–2–3–4–5` impulse** at some higher degree. The decision changes the entire downstream analysis.

Diagnostic checklist (look at the rally on Weekly):
- **In-progress impulse (still inside Wave 3):** rally is mostly vertical, no clean internal `1-2-3-4-5` is visible at the next degree down, no clear intermediate `4` pullback inside the rally.
- **Complete impulse (1–5 done):** the rally itself contains a visible `(1)(2)(3)(4)(5)` (or circled `①②③④⑤`) — i.e. there is at least one significant *pullback inside the rally* that fits the internal Wave 4, followed by another leg up to a final Wave 5 high (often with bearish RSI divergence).

What changes if the impulse is complete:

| Aspect | In-progress Wave 3 | Complete Wave 1–5 |
|--------|--------------------|----|
| Identity of the latest big rally | bigger **Wave 3** of a larger impulse | bigger **Wave 1** (or `3`) of an even higher degree, *now finished* |
| Identity of the next correction | bigger **Wave 4** (shallow, `0.236–0.50`, must respect Rule 3) | bigger **Wave 2** (or `A–B–C` of even higher-degree correction): **deep, `0.5–0.618`** of the *whole* impulse |
| Default Fib draw | impulse leg → expect `0.236–0.382` retrace | full impulse start → end → expect `0.5–0.618` retrace |
| Where is `0.382`? | the BUY ZONE — Wave 4 termination | usually just **sub-`(A)`** of the bigger correction — *not* the final low |
| Where is the BUY ZONE? | `0.236–0.382` of the impulse | **`0.5–0.618`** of the entire impulse |
| Sub-`(B)` of the correction | `B` retraces `0.5–0.618` of `(A)` | same — but at a higher degree |
| Bull target after correction | bigger Wave 5 retest of prior high (`+`mild new high) | new bigger Wave 3 / 5 — `1.618` ext from BUY ZONE |

**Rule:** when the chart shows circled `③④⑤` *inside* the rally, the impulse is **complete** — every count below that must be `(A)–(B)–(C)` (or higher-degree `1–2`), and the `0.382` print is just sub-`(A)`, not the buy. Pulling the trigger on `0.382` in this case = catching the knife.

**Corollary for analysis output:** if the impulse is complete, the BUY ZONE band must be drawn at `0.5–0.618` of the *entire* impulse, not `0.236–0.382`. Surface this explicitly in the output and label `0.382` as **sub-`(A)` waypoint, not BUY**.

**Normal impulse — quick check:** Rules **1** and **3** first; if both hold, confirm impulse and project Wave 5 (then expect ABC). Rule 1 fail → likely still corrective. Rule 3 fail → invalid normal impulse (try ABC); overlap at **Wave 1 / Wave A** → **Leading Diagonal**; wedge + overlap at **Wave 5 / Wave C** → **Ending Diagonal** (see reference).

When checking a possible **Leading Diagonal**, validate context before invalidating: start-of-structure location, sub-wave 4 as a 3-wave pullback into expected Fib zone, and (if used) support above the 200-day MA.

| Rule | Condition |
|------|-----------|
| Rule 1 | Wave 2 never retraces past the start of Wave 1 |
| Rule 2 | Wave 3 not shorter than **both** Wave 1 and Wave 5 |
| Rule 3 | Wave 4 never enters Wave 2 territory |

**Guidelines (common but not absolute):**
- Wave 2 typically retraces 50–61.8% of Wave 1
- Wave 4 typically retraces no more than 38.2% of Wave 3
- Wave 5 is typically the shortest of Waves 1, 3, 5
- In a Zig-Zag correction, Wave B retraces no more than 61.8% of Wave A

When Wave 4 is still forming, keep multiple valid structures active (standard ABC, flat, triangle, running-flat style) and wait for confirmation before forcing a single label.

**Bigger Wave 4 commonly internalises as a sub-`(A)–(B)–(C)` zig-zag/flat** (especially after parabolic Wave 3s). When you label a recent pullback as bigger Wave 4, expect three legs and look for the `(C)` low to land inside the BUY ZONE band defined by 0.236–0.50 of bigger Wave 3. Do **not** label that pullback as bigger Wave 2; that mistake forces a deep retrace bias and misses the bigger Wave 5 that follows. See `graphs/onds-wave3-vs-wave1-case.png`.

### Step 2.5 — Wave-Count Sanity Check (required before Fib levels)

Before drawing any Fibonacci or producing the output, answer these four questions explicitly. If any answer is uncertain, drop confidence and present both interpretations side-by-side (preferred + alternate).

1. **Where did the most recent significant rally start and end?** (e.g. `$88 → $796`).
2. **Inside that rally, is there a visible internal `(1)(2)(3)(4)(5)` or `①②③④⑤`?**
   - **Yes** → the rally is a **complete bigger-wave impulse**. The next move is a **higher-degree correction** (`(A)(B)(C)` or bigger Wave 2). BUY ZONE = `0.5–0.618` of the *entire* rally.
   - **No / not visible** → the rally is **still inside a Wave 3** (or is a Wave 1). The next move is a **shallow Wave 4**. BUY ZONE = `0.236–0.50` of the rally.
3. **Where does the `0.382` of the rally project?** Is the recent low landing **at** `0.382` (Wave 4 candidate) or **just past** it without holding (sub-`(A)` of higher-degree correction)?
4. **Does the bounce off the recent low look impulsive (5 sub-waves up) or corrective (3 sub-waves up)?**
   - 5 sub-waves up → the prior low was a real bottom (Wave 4 or Wave 2 done) — bullish.
   - 3 sub-waves up → the bounce is sub-`(B)` of a still-running correction — **bearish**, expect the next leg down to the deeper BUY ZONE.

Output the answers in one short paragraph just before the BULL/BEAR Fibonacci block. Never skip this step.

### Step 3 — Fibonacci Levels

| Wave | Draw From → To | Zone |
|------|----------------|------|
| Wave 2 end | W1 start → W1 end | 50%–61.8% retrace |
| Wave 3 end | W1 start → W1 end (expand from W2) | 161.8%–200% (extended: **261.8%**) extension |
| Wave 4 end | W3 start → W3 end | 23.6%–38.2% retrace |
| Wave 5 end | W4 end → W4 inverse length | 100%–161.8% (extended: **261.8%**) extension |
| Bigger-degree wave terminal | sub-(1) start → sub-(1) end | **261.8%** extension (sum-of-sub-waves target) |
| ZZ Wave B | A start → A end | 50%–61.8% retrace |
| ZZ Wave C | A start → A end (from B) | 100% (extended: **161.8%**) extension |

**Always include `2.618`** when projecting:
- An **extended Wave 3** (after major news/volatility) — primary target shifts from `1.618` → **`2.618`**
- A **bigger-degree wave terminal** projected from the first sub-wave (e.g. bigger Wave `III` ≈ `2.618` of sub-`(1)`)
- An **extended Wave 5** when Wave 3 was the shorter of 1/3
Never drop the `2.618` print level just because `1.618` looks adequate — it is the next confluence and the standard “runner trim” zone.

Quick impulse guide: W2 often near 0.618 of W1, W3 often near 1.618 of W1 from W2 (extended near 2.618), W4 often near 0.382 of W3, W5 often close to W1 length (extended → 1.618 / 2.618), and the bigger-degree wave terminal often prints near **2.618 of the first sub-wave**.

Full corrective and complex correction tables: see [ewt-algorithm.md](ewt-algorithm.md).

### Position Sizing (risk first)

Apply the position-sizing framework from [ewt-algorithm.md](ewt-algorithm.md) before giving add/trim plans.

- Set risk per idea first (typical baseline: **1–2%** of portfolio)
- Size from invalidation distance, not conviction alone:
  - `risk_dollars = portfolio_value * risk_pct`
  - `position_value = risk_dollars / stop_pct`
  - `shares = risk_dollars / abs(entry - stop)`
- Scale in as confirmation improves (5-part method: breakout hold, retest, Wave 3 progression, Wave 3 confirmation, reserve tranche)
- Do not average down mechanically; prefer adding as winners confirm
- If no hard stop is used, state that clearly and define structural invalidation instead

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

### Step 5.5 — Action Recommendation (required)

Always provide an explicit directional call:
- **BUY** when structure, wave count, and invalidation/risk are favorable now
- **HOLD** when setup is valid but confirmation is still pending
- **SELL / TRIM** when targets are met, momentum fades, or invalidation risk rises

Format:
- `Recommendation: [BUY / HOLD / SELL / TRIM]`
- `Why: [1 short sentence tied to wave/fib/structure]`
- `Condition: [what must happen next to keep/flip recommendation]`

#### Step 5.5a — Add / Trim cue (required, directly under recommendation)

Always emit an **Add / Trim cue** immediately below the recommendation. The cue is a function of **bigger trend wave × current sub-wave** — never derived from price alone. Use the table below; if the situation does not fit, default to **HOLD / no fresh action** and explain why.

Mapping (bigger wave is the primary motive wave in play; sub-wave is one degree below):

| Bigger wave (primary) | Current sub-wave | Add cue | Trim cue |
|-----------------------|------------------|---------|----------|
| `1` / `III` impulse up — early | `(1)` extending up | No new add — chasing risk | None — let it run |
| `1` / `III` impulse up | `(2)` pullback (corrective) | **Primary ADD** at `0.5`–`0.618` of `(1)`; lighter add at `0.382` only with strong rejection | None |
| `1` / `III` impulse up | `(3)` running | Add only on shallow `(iv)` micro-pullbacks; do not chase parabolic extensions | Light trim at `1.618` ext if RSI overheated |
| `1` / `III` impulse up | `(4)` pullback | Light ADD at `0.236`–`0.382` of `(3)`; respect Rule 3 (no entry into `(2)` territory) | None unless `(4)` invalidates |
| `1` / `III` impulse up | `(5)` running | No fresh add | **TRIM in stages** at `1.0`/`1.618` ext; tighten stops on RSI bearish divergence |
| `2` / `IV` correction down | `(A)` down | No add — wait | Trim residual longs into bounces |
| `2` / `IV` correction down | `(B)` bounce up | No add — counter-trend trap | **TRIM** any tactical longs into `0.5`–`0.618` of `(A)` |
| `2` / `IV` correction down | `(C)` down (terminal) | **Stage ADDs** as `(C)` approaches `0.5`–`0.618` of bigger `1` (or `0.236`–`0.382` of bigger `3` if in `IV`); confirm with RSI bull div + bullish reaction | None |
| `5` impulse up — late | `(5)` of `5` | No add | **Aggressive TRIM**; keep only a runner with hard stop |
| Corrective `A` / `B` / `C` up (counter-trend) | any | No add — counter-trend | Trim into Fib resistance of bigger prior down leg |
| Leading or Ending Diagonal in play | any | Reduced size only — overlap rules loosen invalidations | Trim early into the wedge apex |

Required output format (one line each, directly below the recommendation block):
- `Add cue: [zone + Fib + sub-wave reason]` — e.g. `$17.45–$15.85 on sub-(2) 0.50–0.618 retrace of (1) inside bigger III`
- `Trim cue: [zone + Fib + sub-wave reason]` — e.g. `$36.84 on sub-(3) 1.618 ext; second trim at $45.16 sub-(5)`
- If a side is N/A, write `N/A — [reason]` (e.g. `Add cue: N/A — currently inside sub-(1) of bigger III, wait for sub-(2)`).

### Step 5.6 — Bull Case vs Bear Case (required)

Every analysis must present both scenarios side-by-side, anchored on the same chart structure:
- **BULL CASE** — assumes the primary impulsive count holds: list the next sub-wave Fib retrace zones (e.g. `(2)` pullback to 0.382 / 0.5 / 0.618), then projected sub-wave `(3)`, `(4)`, `(5)` targets and the bigger-degree wave target above.
- **BEAR CASE** — assumes the primary count fails: list the deeper retrace levels (typically 0.5 / 0.618 of the prior leg) and the level/condition that flips the count to corrective or to a lower degree.
- Mark the **flip line** explicitly: the price/condition that converts the active read from bull to bear (or vice versa).

### Step 6 — Analysis Output

```
═══════════════════════════════════════════════════
EWT ANALYSIS: [TICKER] | [DATE] | [TIMEFRAME]
═══════════════════════════════════════════════════

⚡ QUICK MAP (always first)
   Current price:    [price]
   Bigger trend:     [e.g. "Wave III of impulse up from [date/price]"]
   Sub-wave (now):   [e.g. "sub-wave (1) of III complete; (2) pending"]
   Phase:            [Motive / Corrective]
   Wave anchors:     [W1 start, W1 end, W2 low or equivalent]
   Next targets:     [Target 1, Target 2, Target 3 + Fib labels]
   Invalidation:     [price / condition]
   Recommendation:   [BUY / HOLD / SELL / TRIM]
   Add cue:          [zone + Fib + sub-wave reason, or "N/A — reason"]
   Trim cue:         [zone + Fib + sub-wave reason, or "N/A — reason"]
   Why/Condition:    [one-line rationale + trigger]

📍 CURRENT PRICE: [price]

🌊 WAVE POSITION
   Primary trend:    [Bullish / Bearish]
   Phase:            [Motive / Corrective]
   Bigger wave:      [e.g. "Wave III up from [date/price]"]                ← bare numeral, no parens
   Sub-wave:         [e.g. "sub-wave (1) complete; (2) pending"]            ← parenthesised
   Lower sub-wave:   [optional, e.g. "(i) of (1) — micro count"]            ← only if relevant
   Degree:           [Primary / Intermediate / Minor]
   Confidence:       [High / Medium / Low] — [brief reason]

📐 FIBONACCI LEVELS
   Wave count drawn from: [start price] → [end price]

   🟢 BULL CASE — sub-(2) pullback then (3)/(4)/(5):
      sub-(2) at: [price] (38.2%) / [price] (50%) / [price] (61.8%)
      sub-(3) target: [price] (1.618 ext) → extended [price] (2.618 ext)
      sub-(4) pullback: [price] (~0.382 of sub-(3))
      sub-(5) target: [price] (1.0–1.618 ext) → extended [price] (2.618 ext)
      Bigger wave target: [price] (2.618 of sub-(1) — sum-of-sub-waves)

   🔴 BEAR CASE — count failure / deeper correction:
      First flip: [price] ([structure broken — e.g. loss of (2) 0.618])
      Deeper retrace: [price] (0.5 / 0.618 of bigger leg)
      Hard invalidation: [price] (bigger Wave II low / Rule 1 break)

   ⚖️  FLIP LINE: [price/condition]

⚠️  INVALIDATION
   [Price/movement that invalidates the count]

📊 RSI DIVERGENCE
   [Present / Absent] — [description]

🔲 CHART PATTERN
   [Pattern] → [Wave match]

💡 TRADE MANAGEMENT SUMMARY
   Current situation: [1-2 sentences]

🧮 POSITION SIZING
   Portfolio size:   [capital]
   Risk per idea:    [1-2% or custom]
   Invalidation:     [price level / % distance]
   Max position:     [value and/or shares]
   Scale-in plan:    [5-part or custom]
   Notes:            [stop-loss based or structural invalidation based]

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

When the user pastes course notes, lessons, or other reference material and asks to update the skill:

1. **Diff first** — read `SKILL.md` and [ewt-algorithm.md](ewt-algorithm.md) and decide what is **genuinely new** vs already covered. Do not duplicate rules, tables, or definitions that already exist. Do not add course **lesson numbers** to the docs (e.g. “Lesson 3”); map content to the existing section names only.
2. **Skip or trim** — if the paste only rephrases existing content, say so briefly and make no edit (or add only the one missing nuance, exception, or invalidation case).
3. **Place once** — detailed EWT theory → `ewt-algorithm.md`; workflow, steps, and agent behaviour → this file. Do not mirror the same table in both files.
4. Use the **note-taker** skill for general programming/tooling findings in `~/GitRepos/book/` (e.g. `python.md`, `rust.md`).
5. Update this `SKILL.md` only when the finding changes how the agent should analyse or build.
6. Update `ewt-algorithm.md` only when wave-counting, rules, patterns, or Fibonacci methodology gains new or corrected detail.
