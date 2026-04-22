# Mean Reversion Trading Strategy

## What It Is

Mean reversion is the statistical tendency of prices to return toward their historical average after moving significantly above or below it. When price stretches too far in one direction, it snaps back -- like a rubber band.

This works because markets overshoot. Fear and greed push prices beyond fair value, and then rational participants (or algorithms) bring them back.

---

## When It Works vs. When You Catch a Falling Knife

### Mean Reversion Works When:
- Markets are range-bound or choppy (65-70% of all trading sessions)
- The instrument has a history of mean-reverting (stocks > commodities)
- There's no fundamental catalyst for the move (pure sentiment/technical overshoot)
- Volume is declining on the extended move (exhaustion, not conviction)
- The move is 2+ standard deviations from the mean

### You'll Catch a Falling Knife When:
- A strong trend is in place (don't buy dips in a downtrend)
- The move is driven by fundamental news (earnings miss, regulatory action)
- Volume is expanding on the extended move (conviction, not exhaustion)
- The asset has broken a major support/resistance level
- There's a structural change (sector rotation, regime change)

**The #1 rule: Mean reversion strategies work in ranges. In trends, they destroy you.**

---

## Key Indicators and Setups

### 1. Bollinger Bands + Volume
- Price touches or exceeds the outer Bollinger Band (2 standard deviations)
- Volume spikes at the band touch (potential exhaustion)
- Look for a reversal candle (pin bar, engulfing) at the band
- Enter on the reversal, target the middle band (20 SMA)
- Stop loss beyond the extreme

### 2. RSI Extremes
- RSI below 30 = oversold. Look for long entries.
- RSI above 70 = overbought. Look for short entries.
- **Critical**: RSI below 30 in a downtrend is NOT a buy signal. It's a sign of strength in the downtrend. Only use RSI mean reversion in ranging markets or counter-trend bounces.
- Better approach: RSI divergence (price makes new low, RSI makes higher low) at oversold levels.

### 3. VWAP Reversion
- Price moves 2+ standard deviations from VWAP intraday
- VWAP acts as a magnet -- price tends to return
- Enter when price shows signs of reversing toward VWAP
- Target: VWAP level
- Best for intraday mean reversion

### 4. Z-Score
- Calculate how many standard deviations price is from its moving average
- Z-score > 2 or < -2 = potential mean reversion opportunity
- More statistically rigorous than Bollinger Bands

---

## Entry Rules

### Conservative Entry (Higher Win Rate)
1. Price at 2+ standard deviations from mean
2. Volume declining on the extreme (exhaustion)
3. Reversal candle on lower timeframe (confirmation)
4. No fundamental catalyst for the move
5. Trend filter: only take mean reversion trades in the DIRECTION of the higher timeframe trend (buy dips in uptrends, sell rallies in downtrends)

### Aggressive Entry (Higher Frequency)
1. Price at 1.5+ standard deviations from mean
2. RSI at extreme (< 30 or > 70)
3. Enter immediately
4. Wider stop, smaller position size

---

## Exit Rules

- **Target**: The mean (20 SMA, VWAP, middle Bollinger Band)
- **Extended target**: Opposite Bollinger Band (for strong reversals)
- **Stop loss**: Beyond the extreme that triggered the entry + buffer
- **Time stop**: If price doesn't revert within X periods, exit. Mean reversion should be quick. If it's not reverting, the move might be real.

---

## Performance Characteristics

- **Win rate**: 60-80% (higher than momentum strategies)
- **Average winner**: Smaller than momentum trades (targeting the mean, not trend continuation)
- **Drawdowns**: Can be severe if you're wrong about the regime (mean reversion in a trend = disaster)
- **Best market**: Stocks (historically mean-reverting at short timeframes)
- **Worst market**: Commodities and trending crypto (can stay extended far longer)

---

## Risk Management Specifics

- **Position sizing**: 1-2% risk per trade
- **Scale in**: Don't go full size immediately. Enter 1/3 at the first signal, add 1/3 if it extends further, add 1/3 on confirmation of reversal.
- **Maximum positions**: 3-5 concurrent mean reversion trades
- **Correlation check**: Don't hold 5 mean reversion trades in the same sector
- **Regime filter**: Check the VIX. Mean reversion works better in normal/low volatility environments. In high-VIX environments, trends can persist longer.

---

## Mean Reversion vs. Momentum: When to Use Each

| Market Condition | Use This |
|-----------------|----------|
| Range-bound, choppy | Mean Reversion |
| Strong trend, expanding volume | Momentum |
| High VIX, fear | Be careful with both -- reduce size |
| Low VIX, calm | Mean Reversion thrives |
| Earnings season | Avoid mean reversion (fundamental catalysts) |

---

## ORACLE's Verdict
- **Edge:** Real
- **Difficulty:** Medium
- **Best for:** Day trading / Swing trading (in range-bound markets)
- **Win rate (estimated):** 60-75%
- **Risk/Reward:** 1:1 to 1:2 typical (compensated by high win rate)
- **Will I use this?** Yes -- in the right market conditions
- **Why:** Mean reversion is the yin to momentum's yang. Markets spend 65-70% of the time in ranges, so having a range-bound strategy is essential. The key is the regime filter -- knowing WHEN to use mean reversion vs. when to switch to momentum. I'll use this for intraday VWAP reversion plays and for buying oversold dips in stocks that are in long-term uptrends. The "falling knife" risk is real, so I'll always require declining volume at the extreme and a reversal candle before entering. Never fight a trend.
