---
tags: [oracle, trading, technical-analysis, timeframe]
created: 2026-04-21
---

# Multi-Timeframe Analysis

## The Core Principle

Never trade on one timeframe alone. The daily chart sets the trend, the 4hr sets the swing, the 1hr sets the entry. All three must agree before entering. Trading a single timeframe is like reading one sentence of a book and guessing the plot — you're operating with incomplete information.

## Why Multiple Timeframes Work

Higher timeframes carry more weight because they represent more aggregated decisions, more capital, and more committed participants. A daily support level held by thousands of traders over weeks is far more significant than a 5-minute level held for 20 minutes. The principle is simple: zoom out to identify what's real, zoom in to execute precisely.

---

## The 3-Screen System (Elder's Method, Adapted for Modern Markets)

Alexander Elder's original system used weekly, daily, and intraday charts. Adapted for active trading in 2026:

### Screen 1 — Daily Chart (The Tide)
**Question: What is the primary trend?**

Use the 50-day EMA and 200-day SMA as primary trend filters.
- Price **above both** = primary uptrend. Only look for long setups.
- Price **below both** = primary downtrend. Only look for short setups.
- Price **between** them = no man's land. Reduce size, avoid large conviction trades.
- 50 EMA crossing above 200 SMA = Golden Cross (confirmed bull regime).
- 50 EMA crossing below 200 SMA = Death Cross (confirmed bear regime).

Additional daily tools: MACD direction (not level), volume trend (rising on up days = healthy), ADX > 25 = trending, ADX < 20 = ranging (don't use trend-following tactics).

### Screen 2 — 4-Hour Chart (The Wave)
**Question: Within the primary trend, where is the current swing?**

In an uptrend, you want to buy pullbacks to support — not chase extensions. In a downtrend, you want to sell bounces to resistance — not short into oversold conditions.

Key tools:
- **RSI(14):** In an uptrend, RSI pulling back to 40-50 zone = buy zone. RSI above 70 = extended, wait. In a downtrend, RSI bouncing to 50-60 = short zone. RSI below 30 = oversold bounce risk.
- **VWAP anchored from swing high/low:** Price below anchored VWAP after a pullback = still weak. Reclaim of anchored VWAP = potential reversal.
- **Fibonacci retracements:** 38.2%, 50%, 61.8% are natural pullback targets in a trend. The 50% retracement is historically the most reliable entry in a healthy trend.

### Screen 3 — 1-Hour or 15-Min Chart (The Ripple)
**Question: Right now, is there a specific trigger to enter?**

This is not where you decide direction — Screen 1 and 2 decide that. Screen 3 is purely timing. You're waiting for one of these triggers:
- **Bullish engulfing or hammer candle** at the 4hr support level
- **VWAP reclaim with volume** (price breaks above VWAP and holds for 2 candles)
- **Volume spike** on the reversal candle (1.5x+ above the 20-period average volume)
- **Break and retest:** Price breaks above a key level, pulls back to test it as support, then bounces
- **Opening range breakout (ORB):** First 15-30 minutes of session establishes range; trade the break with volume confirmation

---

## Alignment Table

| Daily Trend | 4hr Position | 1hr Trigger | Action | Notes |
|-------------|-------------|-------------|--------|-------|
| Bullish (above 50/200) | Pulling back to support (RSI 40-50) | Hammer or engulfing with volume | **BUY — full size** | All three screens aligned |
| Bullish | Pulling back to support | Still declining, no reversal candle | **WAIT** | Right setup, wrong timing |
| Bullish | Bullish (RSI 65-70+) | Bullish | **NO ENTRY — chasing** | Extended, wait for pullback to Screen 2 support |
| Bullish | At key support | Trigger present | **BUY — normal size** | Valid, but confirm daily trend is solid |
| Bearish (below 50/200) | Bouncing to resistance (RSI 50-60) | Rejection candle, volume decline | **SHORT — full size** | All three screens aligned |
| Bearish | Bouncing | Still rising, no rejection | **WAIT** | Right setup, wrong timing |
| Mixed / choppy | Any | Any | **NO TRADE** | Trend not clear — sit on hands |
| Ranging (ADX < 20) | Any | Any | **REDUCE SIZE or no trade** | Mean reversion only, no trend-following |

---

## How ORACLE Documents Every Trade Entry

Every trade requires completing this checklist before entering:

```
MULTI-TIMEFRAME ENTRY CHECKLIST
================================
Ticker: ____        Date/Time: ____

[ ] Daily trend: BULL / BEAR / CHOPPY
    50 EMA vs 200 SMA: ____
    Daily MACD direction: ____
    ADX reading: ____ (>25 = trending, <20 = ranging)

[ ] 4hr swing position:
    Where in the swing: PULLBACK / EXTENDED / AT RESISTANCE
    4hr RSI: ____
    Key 4hr support/resistance level: ____

[ ] 1hr entry trigger:
    Trigger type: ____
    Volume confirmation: YES / NO (ratio vs average: ____x)
    Trigger candle time: ____

ALIGNMENT: FULL (3/3) / PARTIAL (2/3) / NONE (0-1/3)

If not FULL alignment → NO TRADE or HALF SIZE max
```

---

## Common Alignment Mistakes

**Mistake 1: Trading Screen 3 in isolation.**
A bullish 1hr pattern means nothing if the daily is in a downtrend. You're fighting the tide.

**Mistake 2: Confusing "in pullback" with "reversing."**
A 4hr that is still declining is a pullback continuing, not a reversal. Wait for the 1hr trigger to confirm the bounce has actually started.

**Mistake 3: Being impatient at the Screen 2 level.**
Price reaches the 4hr support zone and you enter immediately, but the pullback continues for 2 more days before reversing. The 1hr trigger exists precisely to avoid this. Wait for the candle.

**Mistake 4: Treating all timeframe alignments equally.**
A daily trend that has lasted 6 months is more powerful than one that started 2 weeks ago. Scale position size with the strength and duration of the primary trend.

---

## Crypto Adaptation (24/7 Markets)

Crypto has no daily open/close, no overnight gaps, no pre-market dynamics. The timeframe principle still applies — it just runs continuously.

| Traditional | Crypto Equivalent | Purpose |
|------------|------------------|---------|
| Weekly chart | Daily chart | Macro trend |
| Daily chart | 4hr chart | Swing direction |
| 4hr chart | 1hr chart | Momentum |
| 1hr chart | 15-min chart | Entry trigger |

**Crypto-specific notes:**
- Weekend sessions often have lower liquidity — 4hr patterns are less reliable. Prefer daily-level analysis for weekend trades.
- BTC dominance acts as a "meta-screen": if BTC is declining in dominance, altcoins may be in relative uptrend even if BTC is flat.
- Major support/resistance on BTC daily chart affects ALL crypto. Check BTC alignment before trading altcoins.
- Funding rates on perpetuals act as a sentiment indicator: highly positive funding = crowded long (contrarian bearish signal at resistance). Highly negative = crowded short (contrarian bullish signal at support).

---

## Quick Reference: Timeframe Hierarchy

```
TIMEFRAME WEIGHT
================
Daily    ████████████████████  100% — sets direction, do not fight
4hr      ████████████████      80%  — sets entry zone
1hr      ████████████          60%  — provides trigger timing
15min    ████████              40%  — fine-tune, stop placement only
5min     ████                  20%  — noise, do not use for decisions
```

The higher the timeframe agreement, the larger the position size. Trade with the tide, not against it.
