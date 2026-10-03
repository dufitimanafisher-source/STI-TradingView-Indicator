# XAUUSD Test Checklist (1M / 5M)

## Setup & Environment

- [ ] Symbol: **XAUUSD** (or your forex/gold broker pair)
- [ ] Timeframes: **1M and 5M only**
- [ ] Bar Replay: **ON** (TradingView Pro feature)
- [ ] Strategy Tester: **ON** (if available)
- [ ] Indicator: **STI** loaded with default inputs

## Code Compilation & Syntax

- [ ] Script compiles cleanly in Pine Script v6 editor (no CE errors)
- [ ] No deprecated function warnings
- [ ] No variable naming conflicts
- [ ] Alert messages format correctly (no quote escaping issues)

## No-Repainting Verification

- [ ] All `request.security()` calls use `lookahead=barmerge.lookahead_off`
- [ ] All HTF data shifted by `[1]` (not current bar)
- [ ] Entry signals fire only when `barstate.isconfirmed == true`
- [ ] No price array access beyond 1–2 bars (no future lookahead)
- [ ] Bar replay: chart does not redraw previous signals as price moves
- [ ] Alert fires once per bar close, not on each tick

## Chart Output

- [ ] Only **4 objects per trade**:
  1. Green/red entry line (solid, width 2)
  2. BUY/SELL label above/below entry
  3. Gray dashed SL line with price
  4. Teal/orange dotted TP line with price
- [ ] **No zones, boxes, or pattern overlays**
- [ ] **No extra labels** (e.g., entry names, structure tags)
- [ ] Entry line extends right (extend.right)
- [ ] SL and TP lines extend right (extend.right)
- [ ] Label color: green BUY, red SELL
- [ ] All objects placed at bar_index of signal

## Timeframe Warnings

- [ ] Chart > 5M displays warning label: "STI: Use 1M/5M only"
- [ ] Warning alert fires once per bar on higher timeframe
- [ ] Warning appears in top-left corner with orange background
- [ ] 1M and 5M charts do not show warning

## Entry Signal Logic

### Bias Confirmation
- [ ] BUY only fires if bullish bias detected (HH/HL or EMA slope or HTF close > prev)
- [ ] SELL only fires if bearish bias detected (LH/LL or EMA slope or HTF close < prev)
- [ ] No signal if bias is neutral or opposing

### Confirmation Candle Strength
- [ ] Body ≥ 50% of range (confirmationBodyRatio input)
- [ ] Close beyond prior high (BUY) or below prior low (SELL)
- [ ] Weak candles (small body or inside prior range) do not signal

### RR Filter
- [ ] SL calculated first (HTF support/resistance + ATR buffer)
- [ ] TP calculated from entry +/- RR multiple of SL distance
- [ ] If RR < 1.5 (minRR input), signal is **suppressed**
- [ ] BUY signal example: entry=1950, SL=1945, TP=1960 → RR=3 ✓

### Six Entry Rules (by type)

#### 1. Breakout
- [ ] Fires when: close > range high AND close > open AND (close-open) > (high-close) AND bullish bias
- [ ] Visual: candle body large, wick small in breakout direction
- [ ] No signal if range compressed or breakout already extended

#### 2. Pullback/Retest
- [ ] Fires when: close overlaps last swing AND close > lower EMA (20/50) AND bullish bias
- [ ] Retracement into zone, no break of last swing
- [ ] Rejection candle (pin or engulf) implied by close > prior high

#### 3. Reversal
- [ ] Fires when: close > highest(5) AND close > close[1] AND bullish bias
- [ ] CHoCH (close change of character) logic implemented
- [ ] Stricter than impulse-mode signals

#### 4. S/R Bounce
- [ ] Fires when: close < highest(20) AND close > lowest(20) AND (high-close) >= 2×body AND bullish bias
- [ ] Wick ≥ 2× body (rejection)
- [ ] Close back outside the zone

#### 5. Liquidity Sweep
- [ ] Fires when: close > prevDayHigh AND close < prevDayHigh + ATR AND close < close[1] AND bullish bias
- [ ] Sweep logic: spike beyond, reclaim inside
- [ ] Suppressed if strong follow-through (real breakout)

#### 6. Continuation
- [ ] Fires when: close > highest(high[1], 10) AND close > close[1] AND bullish bias
- [ ] Pole + pause + break in same direction
- [ ] Strong impulse continuation

## Cooldown & One-Trade-at-a-Time

- [ ] After signal fires, no new signal for N bars (cooldownBars input, default 10)
- [ ] If one open trade exists, no new signal until it closes
- [ ] After signal close, cooldown counter resets
- [ ] `lastTradeBar` variable updated on each signal

## Stop & Take-Profit Levels

### Stop Loss
- [ ] SL for BUY = low - (ATR × buffer multiplier) - tolerance
- [ ] SL for SELL = high + (ATR × buffer multiplier) + tolerance
- [ ] SL respects minSL (tightest: ATR × 0.5) and maxSL (widest: ATR × 6.0)
- [ ] SL is always beyond the signal candle
- [ ] SL line drawn at correct price on chart

### Take-Profit
- [ ] TP for BUY = entry + (entry - SL) × RR multiple
- [ ] TP for SELL = entry - (SL - entry) × RR multiple
- [ ] TP drawn only if RR ≥ 1.5
- [ ] TP line extends right, marked with price

## Alert Messages

- [ ] BUY alert: `BUY Entry=<price> SL=<price> TP=<price>`
- [ ] SELL alert: `SELL Entry=<price> SL=<price> TP=<price>`
- [ ] Prices formatted with correct decimal places
- [ ] Alerts fire once per bar close
- [ ] Alerts appear in TradingView notification panel

## HTF Bias Confirmation (request.security)

- [ ] 1D close > 1D prev high → bullish bias
- [ ] 1D close < 1D prev low → bearish bias
- [ ] 4H close > 4H prev high → bullish bias (agrees with 1D)
- [ ] Local trend: close > EMA(20) AND close > EMA(50) → bullish
- [ ] No signal if HTF bias disagrees with entry direction

## Liquidity & Level Logic

### Levels Detected
- [ ] Previous day high/low (prevDayHigh, prevDayLow)
- [ ] Swing highs/lows over rangeLen bars
- [ ] Local EMA levels (20, 50)
- [ ] Equal highs/lows (repeated levels)

### Tolerance
- [ ] Price within levelTol of level treated as touching level
- [ ] Default 0.0 (exact touch)
- [ ] Can be increased for noise reduction

## Performance & Edge Cases

- [ ] No script errors on cold start (first 50 bars)
- [ ] No lag on real-time bar updates
- [ ] Script handles low liquidity assets (sparse data)
- [ ] Division by zero guarded (syminfo.mintick in RR calc)
- [ ] `na()` checks prevent referencing undefined values
- [ ] Max lines (50), max labels (50), max boxes (50) not exceeded

## Test Run: Bar Replay Session (10–20 mins)

1. **Load chart** XAUUSD 5M with STI indicator
2. **Enable bar replay** (right-click chart → Bar Replay)
3. **Play back** to a recent well-formed structure (e.g., breakout or pullback from 1D resistance)
4. **Observe**:
   - [ ] Signal fires on close of confirmation candle (not mid-bar)
   - [ ] Entry, SL, TP lines appear together
   - [ ] BUY/SELL label correct color and position
   - [ ] Alert fires in notification panel
5. **Check SL/TP**:
   - [ ] SL is logical (beyond breakout level or swing)
   - [ ] TP respects RR minimum
   - [ ] No SL/TP inside opposite HTF level
6. **Switch to 1M** and observe the same setup:
   - [ ] Signal fires at same area (multi-TF alignment)
   - [ ] No warning on 1M
   - [ ] Same bias confirmation across timeframes

## Test Run: Strategy Tester (optional, for backtesting accuracy)

1. **Open Strategy Tester** for XAUUSD (or convert STI to strategy)
2. **Set**: 1M/5M, date range (1–3 months), default inputs
3. **Run backtest**:
   - [ ] No repainting detected (results stable on second run)
   - [ ] Trades enter on confirmed bar close only
   - [ ] SL and TP values consistent with rules
   - [ ] Win rate reasonable (>40% expected, quality over quantity)
4. **Review P&L**:
   - [ ] Majority of wins occur on high-RR trades (RR > 2)
   - [ ] Few losses cluster in low-RR trades
   - [ ] Equity curve smooth (no sudden spikes = no lookahead)

## Manual Trade Review (Signal Quality)

After 10–20 signals in bar replay, rate each:

| Signal # | Type | Entry | SL | TP | Bias | Body% | RR | Quality | Note |
|----------|------|-------|----|----|------|-------|----|---------|----|
| 1 | Breakout | 1950.5 | 1948 | 1958 | Bull | 65% | 2.8 | ✓ Good | Clear HTF align |
| 2 | Pullback | 1960 | 1957 | 1967 | Bull | 52% | 2.3 | ✓ Good | Strong rejection |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

Look for:
- [ ] No false signals on noise or choppy price
- [ ] Signals cluster around major levels
- [ ] Each signal follows complete entry rule
- [ ] Bias always matches entry direction
- [ ] Confirmation candle noticeably strong

## Final Checklist

- [ ] All six entries working and firing correctly
- [ ] No repainting on bar replay
- [ ] No alerts on old bars after reload
- [ ] Chart output matches spec (4 objects, no extras)
- [ ] Warnings fire only on chart > 5M
- [ ] HTF bias gating effective (rejects disagreeing bias)
- [ ] RR filter suppresses marginal trades
- [ ] Cooldown prevents over-trading
- [ ] Alert messages format and fire once per bar
- [ ] Manual review confirms high signal quality

## Sign-Off

- [ ] Tested on: **XAUUSD** 1M/5M
- [ ] Date: **_______________**
- [ ] Tester: **_______________**
- [ ] Result: **✓ PASS / ✗ FAIL**
- [ ] Notes: **_______________**

---

If any test fails, check the Pine Script console for errors, review the relevant logic section in STI.pine, adjust inputs, and re-run.
