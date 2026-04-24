---
type: trading-tool
agent: ORACLE
title: Charting Protocol
created: 2026-04-21
last_updated: "{{date}}"
---

# Charting Protocol

**ORACLE does not trade blind. Every session, charts are checked. Levels are maintained. Patterns are identified. This document defines exactly how ORACLE reads and acts on technical analysis.**

---

## Every-Session Chart Check

For each open position, ORACLE evaluates the following. This is not optional. A position without a current chart read is a position without context.

### Daily Chart Assessment (per position)

| Question | How to Answer | Action If Concerning |
|----------|--------------|---------------------|
| Where is price vs 50 SMA? | Above = bullish bias. Below = caution. | If price crosses below 50 SMA → tighten stop, reassess thesis |
| Where is price vs 200 SMA? | Above = long-term uptrend intact. Below = bearish regime. | If price crosses below 200 SMA → strong sell signal unless thesis is extraordinary |
| 50/200 SMA relationship? | 50 above 200 = golden cross (bullish). 50 below 200 = death cross (bearish). | Death cross on a held position → exit unless holding for specific catalyst |
| Trend structure? | Higher highs + higher lows = uptrend. Lower highs + lower lows = downtrend. | Trend break (lower low in uptrend) → reassess immediately |
| Key support/resistance? | Identify the nearest horizontal level above and below current price from recent swing highs/lows | Price approaching support → prepare to add. Price approaching resistance → prepare to take profit. |
| Volume trend? | Is volume increasing on up moves and decreasing on down moves? (bullish) Or the reverse? (bearish) | Volume divergence (price up, volume down) → move may not sustain, tighten stop |
| RSI reading? | Above 70 = overbought. Below 30 = oversold. 40-60 = neutral. | RSI > 80 → take partial profits. RSI < 25 → look for reversal entry if thesis supports. |
| VWAP (intraday)? | Price above VWAP = institutional buyers dominant. Below = sellers. | Below VWAP for 2+ hours → short-term bearish, reduce intraday exposure |

### Session Chart Check Template

Run this every session for all open positions:

```
CHART CHECK: {{date}} {{time}}

| Ticker | Price | vs 50 SMA | vs 200 SMA | Trend | RSI | Volume | Key Level Near | Action |
|--------|-------|-----------|-----------|-------|-----|--------|---------------|--------|
| META   |       |           |           |       |     |        |               |        |
| AAVE   |       |           |           |       |     |        |               |        |
| BTC    |       |           |           |       |     |        |               |        |
| RENDER |       |           |           |       |     |        |               |        |
| AMZN   |       |           |           |       |     |        |               |        |
| SUI    |       |           |           |       |     |        |               |        |
| NVDA   |       |           |           |       |     |        |               |        |
```

---

## Key Levels Table

ORACLE maintains this for every open position AND every HOT/WARM watchlist name. Updated weekly (or immediately when a level is broken).

### Open Positions — Key Levels

| Ticker | Support 1 | Support 2 | Resistance 1 | Resistance 2 | 50 SMA | 200 SMA | Trend | Last Updated |
|--------|-----------|-----------|-------------|-------------|--------|---------|-------|-------------|
| META | | | | | | | | |
| AAVE | | | | | | | | |
| BTC | | | | | | | | |
| RENDER | | | | | | | | |
| AMZN | | | | | | | | |
| SUI | | | | | | | | |
| NVDA | | | | | | | | |

### Watchlist — Key Levels

| Ticker | Support 1 | Support 2 | Resistance 1 | Resistance 2 | 50 SMA | 200 SMA | Trend | Last Updated |
|--------|-----------|-----------|-------------|-------------|--------|---------|-------|-------------|
| ETH | | | | | | | | |
| SOL | | | | | | | | |
| GS | | | | | | | | |
| PLTR | | | | | | | | |
| CRWD | | | | | | | | |
| RKLB | | | | | | | | |
| IWM | | | | | | | | |

### How to Update Key Levels
1. **Support levels:** Look left on the daily chart. Find the two most recent swing lows where price bounced. Those are S1 and S2.
2. **Resistance levels:** Find the two most recent swing highs where price stalled. Those are R1 and R2.
3. **Moving averages:** Record the current 50 SMA and 200 SMA values. These move every day.
4. **Trend:** UP (higher highs + higher lows), DOWN (lower highs + lower lows), SIDEWAYS (range-bound), or TRANSITION (breaking out of one into another).

### When Levels Break
- **Support broken:** Support becomes resistance. Move old S1 to R1, find the next support below.
- **Resistance broken:** Resistance becomes support. Move old R1 to S1, find the next resistance above.
- **This is the most important concept in technical analysis.** Failed levels flip their role.

---

## How to Use TradingView MCP

ORACLE should query TradingView every session for systematic technical data. This removes subjectivity from chart reading.

### Per-Position Queries
For each open position, request:
- **Technical rating:** Strong Buy / Buy / Neutral / Sell / Strong Sell (aggregate of multiple indicators)
- **RSI (14):** Current value
- **MACD:** Signal line crossover direction (bullish cross = MACD above signal, bearish = below)
- **Moving averages:** 20, 50, 100, 200 SMA values and whether price is above/below each
- **Bollinger Bands:** Is price near upper band (overbought), lower band (oversold), or middle?
- **ATR (14):** Average True Range — volatility measure. Use for stop placement (stop = entry - 2x ATR)
- **Volume vs average:** Is today's volume above or below the 20-day average?

### Market-Wide Screener Queries
Run these weekly to find new opportunities:

**Momentum Screener:**
- RSI between 50-70 (strong but not overbought)
- Price above 50 SMA
- Volume above 1.5x 20-day average
- ADX above 25 (trending)

**Oversold Bounce Screener:**
- RSI below 30
- Price at or near 200 SMA support
- Volume spike (2x+ average)
- In a sector ORACLE is bullish on

**Overbought Warning Screener (for open positions):**
- RSI above 75
- Price more than 2 standard deviations above 20 SMA
- Volume declining on up days
- Use this to identify positions that need profit-taking

### TradingView Data Logging
After each query, log the key readings:
```
TRADINGVIEW CHECK: {{date}}
| Ticker | Rating | RSI | MACD | vs 50 SMA | vs 200 SMA | ATR | Volume vs Avg |
|--------|--------|-----|------|-----------|-----------|-----|---------------|
```

---

## Chart Patterns — What to Watch For

When scanning charts (daily and 4H timeframes), flag these patterns. Each has a specific probability edge and a defined trade plan.

### Bullish Patterns (look for these when seeking long entries)

**Bull Flag**
- What it looks like: Strong move up (the "pole"), followed by a tight consolidation that drifts slightly down or sideways (the "flag")
- Confirmation: Breakout above the flag on increasing volume
- Target: Measure the pole length. Add it to the breakout point.
- Reliability: High (65-70% success rate in trending markets)
- Action: Enter on breakout above flag. Stop below flag low.

**Double Bottom (W Pattern)**
- What it looks like: Price drops to a support level, bounces, drops back to the same level, bounces again
- Confirmation: Break above the middle peak (the "neckline") on volume
- Target: Measure the distance from bottom to neckline. Add it to breakout point.
- Reliability: High (70%+ when volume confirms)
- Action: Enter on neckline break. Stop below the double bottom.

**Cup and Handle**
- What it looks like: Rounded bottom (the "cup") followed by a small pullback (the "handle")
- Confirmation: Breakout above the handle on volume
- Target: Depth of the cup added to the breakout point
- Reliability: Very high (one of the most reliable bullish patterns)
- Action: Enter on handle breakout. Stop below handle low.

**Ascending Triangle**
- What it looks like: Flat resistance on top, rising support on bottom (higher lows into a ceiling)
- Confirmation: Breakout above resistance on volume
- Target: Height of the triangle added to breakout point
- Reliability: High in uptrends (75%+)
- Action: Enter on breakout. Stop below the last higher low.

### Bearish Patterns (look for these to protect existing longs or find shorts)

**Head and Shoulders**
- What it looks like: Three peaks — left shoulder, higher head, right shoulder at roughly the same level as left
- Confirmation: Break below the neckline (connecting the two troughs) on volume
- Target: Distance from head to neckline, projected downward from breakout
- Reliability: Very high bearish signal (one of the most reliable reversal patterns)
- Action: If holding a long in this name → EXIT. Neckline break is not a "maybe."

**Descending Triangle**
- What it looks like: Flat support on bottom, falling resistance on top (lower highs into a floor)
- Confirmation: Break below support on volume
- Reliability: High bearish signal in downtrends
- Action: If we hold this name → tighten stop to just above the flat support

**Bear Flag**
- What it looks like: Sharp move down (pole), then a tight consolidation drifting slightly up (flag)
- Confirmation: Break below the flag on volume
- Action: Do NOT buy this "discount." This is continuation, not reversal.

### Neutral/Caution Patterns

**Rising Wedge (bearish despite upward slope)**
- What it looks like: Price making higher highs and higher lows, but the channel is narrowing
- Why bearish: Momentum is fading. Each high is less convincing.
- Action: If we hold this name → set trailing stop. Breakdown from wedge is often violent.

**Symmetrical Triangle**
- What it looks like: Converging support and resistance. Could break either way.
- Action: Wait for the break. Do not guess. Enter in the direction of the breakout with confirmation.

---

## Multi-Timeframe Analysis Protocol

Never make a decision on a single timeframe. Use this hierarchy:

| Timeframe | Purpose | What to Look For |
|-----------|---------|-----------------|
| Weekly | Determine the primary trend | Is the trend up, down, or sideways? Where are the major levels? |
| Daily | Find the setup | Is price at a key level? Is a pattern forming? What does RSI say? |
| 4-Hour | Refine the entry | Confirm the daily setup with LTF structure. Look for CHoCH/BOS. |
| 1-Hour | Time the entry | Final confirmation. Enter when 1H structure aligns with daily setup. |

**Rules:**
1. Never trade against the weekly trend unless you have extraordinary evidence
2. The daily chart is the decision timeframe — this is where setups live
3. The 4H and 1H are for TIMING, not for changing the thesis
4. If 4H/1H conflict with daily → wait. Do not force the entry.

---

## Weekly Levels Update Checklist

Every Sunday evening or Monday morning, update all key levels:

- [ ] Update 50 SMA and 200 SMA for all positions and watchlist names
- [ ] Identify any new swing highs/lows formed in the past week
- [ ] Check for any level breaks (support broken = now resistance, and vice versa)
- [ ] Note any chart patterns forming or completing
- [ ] Update the Key Levels Table above
- [ ] Flag any position where price is within 3% of a major level (support, resistance, or moving average)
