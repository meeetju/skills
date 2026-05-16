# EWT Algorithm — Full Reference

Full Elliott Wave Theory framework as taught in *Elliott Waves Made Simple* by Steve Sinclair.
Referenced by `SKILL.md` — load when wave-counting detail or Fibonacci tables are needed.

---

## The Motive Phase (moves with the trend)

### Impulsive Wave (5-3-5-3-5)

Five waves numbered 1–5. Waves 1, 3, 5 move with trend; Waves 2, 4 correct.

**Hard rules (never violated):**

| Rule | Uptrend | Downtrend |
|------|---------|-----------|
| Rule 1 | Wave 2 never falls below the START of Wave 1 | Wave 2 never rises above the START of Wave 1 |
| Rule 2 | Wave 3 is NEVER the shortest of Waves 1, 3, 5 | Same |
| Rule 3 | Wave 4 never enters Wave 2 territory | Same |
| Rule 4 | One of Waves 1, 3, 5 MUST be extended | Same |

Internal structure: Waves 1, 3, 5 → 5 sub-waves each. Waves 2, 4 → 3 sub-waves each.

### Extended Waves (9-wave: 5-3-5-3-5-3-5-3-5)

One of Waves 1, 3, or 5 extends into a 9-wave structure. Most common: **Wave 3**.

### Leading Diagonal (5-3-5-3-5)

- Found ONLY at the very start of a move: Wave 1 or Wave A
- Waves 2 and 4 overlap into Wave 1/A territory
- After a Leading Diagonal → expect a deep pullback (3-wave correction)
- Signals trend is beginning

Rules:
- Wave 2 never falls below Wave 1 start
- Wave 3 is often the longest, never the shortest
- Wave 4 must hold above the end of Wave 2

### Ending Diagonal (3-3-3-3-3)

- Found ONLY at the end of a move: Wave 5 or Wave C
- All sub-waves are 3-wave structures
- Wave 4 enters Wave 2 territory (overlapping — distinguishes from Impulsive)
- Visually looks like a converging wedge
- After an Ending Diagonal → expect a sharp reversal

Rules:
- Wave 2 never falls below Wave 1 start
- Wave 3 is often the longest, never the shortest
- Wave 4 must hold above the end of Wave 2 (uptrend) / below (downtrend)

---

## The Corrective Phase (moves against the trend, labeled A-B-C)

### Simple Corrections

#### Zig-Zag (5-3-5)

Most common correction.
- Wave A: 5 sub-waves (motive)
- Wave B: 3 sub-waves; ends at 50–61.8% retracement of Wave A
- Wave C: 5 sub-waves; breaks beyond the end of Wave A

Rules (uptrend / correction downward):
- Wave B must end BELOW the start of Wave A
- Wave C must break BELOW the end of Wave A

Rules (downtrend / correction upward):
- Wave B must end ABOVE the start of Wave A
- Wave C must break ABOVE the end of Wave A

#### Flat Correction (3-3-5)

All three variants: Wave B ends near 78.6%–90% of Wave A (minimum).

| Type | Wave B | Wave C |
|------|--------|--------|
| Regular Flat | 78.6%–90% of A (doesn't exceed start of A) | Ends near end of A |
| Expanding Flat | Exceeds start of A (123.6%–161.8%) | Exceeds end of A |
| Running Flat | Exceeds start of A; C does NOT reach end of A | Weak — strong underlying trend |

#### Triangle Correction (3-3-3-3-3) — Labeled A-B-C-D-E

Found in Wave 4 (Motive) or Wave B (Corrective). Five sub-waves, each a 3-wave structure.

| Type | Rule |
|------|------|
| Contracting | Each leg smaller than previous (B < A, C < B, D < C, E < D) |
| Expanding | Each leg larger than previous (B > A, C > B, D > C, E > D) |
| Barrier | B and D end at similar horizontal level |
| Running | B briefly spikes beyond A (fundamental event spike) |

After a Triangle → expect a **thrust** in the direction of the prior trend.

### Complex Corrections

#### Double Three (3-3-3) — W-X-Y

Three connected simple corrections separated by X waves.
Each of W, X, Y = Zig-Zag, Flat, or Triangle.

Rules (downtrend):
- Wave X ends above the start of W
- Wave Y ends below the end of W

Rules (uptrend):
- Wave X ends below the start of W
- Wave Y ends above the end of W

**Simple trick:** A Double Three in a downtrend produces 7 swings. Count Higher Highs and Higher Lows from the correction start to identify W-X-Y.

#### Triple Three (3-3-3-3-3) — W-X-Y-X2-Z

Five connected three-wave corrections. Very rare. Label only when simpler alternatives don't fit.

---

## Fibonacci Levels

### Tools

1. **Fibonacci Retracement** — drawn from START of a wave to its END → finds where the NEXT correction ends
2. **Fibonacci Expansion (Projection)** — drawn from START → END of swing → PULLBACK point → finds where the NEXT impulse ends

### Key Levels

```
Retracement: 23.6%, 38.2%, 50.0%, 61.8%, 78.6%, 90%
Extension:   100%, 123.6%, 161.8%, 200%, 261.8%
```

### Motive Phase Targets

| Wave | Draw Fib From → To | Likely End Zone |
|------|-------------------|-----------------|
| Wave 2 end | W1 start → W1 end | 50%–61.8% retrace of W1 (min 38.2%) |
| Wave 3 end | W1 start → W1 end (expand from W2 low) | 161.8%–200% extension of W1 (min 100%) |
| Wave 4 end | W3 start → W3 end | 23.6%–38.2% retrace of W3 |
| Wave 5 end | W4 end → inverse of W4 length | 100%–161.8% inverse extension of W4 |

### Corrective Phase Targets

| Wave | Draw Fib From → To | Likely End Zone |
|------|-------------------|-----------------|
| ZZ Wave B | A start → A end | 50%–61.8% retrace |
| ZZ Wave C | A start → A end (from B) | 100% extension (= end of A) |
| Regular Flat B | A start → A end | 78.6%–90% |
| Expanding Flat B | A start → A end | 123.6%–161.8% |
| Triangle waves | Each from prior swing | 61.8% of prior wave |
| Complex W | Prior impulse | 50%–61.8% of prior impulse |
| Complex X | W start → W end | 50%–78.6% of W |
| Complex Y | W start → W end (from X) | 100%–123.6% of W |

---

## EWT Algorithm Pseudocode (Future Rust Implementation)

```pseudocode
FUNCTION analyse_ticker(ticker: String) -> EWTAnalysis:

  // 1. Fetch OHLCV data (Weekly, Daily, 4H)
  data_w  = fetch_ohlcv(ticker, "1W", bars=200)
  data_d  = fetch_ohlcv(ticker, "1D", bars=200)
  data_4h = fetch_ohlcv(ticker, "4H", bars=200)

  // 2. Find significant turning points (swing highs/lows)
  swings = find_swings(data_w, lookback=10)

  // 3. Determine trend direction
  trend = classify_trend(swings)  // BULLISH | BEARISH | SIDEWAYS

  // 4. Attempt Motive wave count from last major swing low/high
  motive_count = attempt_impulse_count(data_d, swings, trend)

  // 5. Validate against hard rules
  IF NOT validate_impulse_rules(motive_count):
    motive_count = attempt_diagonal_count(data_d, swings, trend)

  // 6. If motive count fails, attempt corrective count
  IF NOT motive_count.valid:
    corrective_count = attempt_corrective_count(data_d, swings, trend)
    current_count = corrective_count
  ELSE:
    current_count = motive_count

  // 7. Calculate Fibonacci levels
  fibs = calculate_fibonacci_levels(current_count)

  // 8. Check RSI divergence
  rsi        = calculate_rsi(data_4h, period=14)
  divergence = detect_divergence(data_4h.close, rsi, current_count.wave_number)

  // 9. Check chart patterns
  pattern = detect_chart_pattern(data_d, current_count)

  // 10. Generate output
  RETURN EWTAnalysis {
    ticker,
    wave_position:    current_count,
    fibonacci_levels: fibs,
    rsi_divergence:   divergence,
    chart_pattern:    pattern,
    add_zones:        fibs.retracements,
    trim_zones:       fibs.extensions,
    stop_loss:        current_count.invalidation_level,
    confidence:       calculate_confidence(current_count, divergence, pattern)
  }

FUNCTION validate_impulse_rules(count: WaveCount) -> bool:
  wave2_low > wave1_start          // Rule 1: W2 never below W1 start
  wave3 != shortest_of(w1, w3, w5) // Rule 2: W3 never shortest
  wave4_high < wave2_high          // Rule 3: W4 never in W2 territory

FUNCTION calculate_fibonacci_levels(count: WaveCount) -> FibLevels:
  // Retracements (corrective wave end)
  retrace_382 = wave_start + (wave_range * 0.382)
  retrace_50  = wave_start + (wave_range * 0.500)
  retrace_618 = wave_start + (wave_range * 0.618)
  retrace_786 = wave_start + (wave_range * 0.786)

  // Extensions (impulsive wave target)
  extend_100  = wave_end + (wave_range * 1.000)
  extend_1236 = wave_end + (wave_range * 1.236)
  extend_1618 = wave_end + (wave_range * 1.618)
  extend_200  = wave_end + (wave_range * 2.000)

  RETURN FibLevels { retracements, extensions }
```
