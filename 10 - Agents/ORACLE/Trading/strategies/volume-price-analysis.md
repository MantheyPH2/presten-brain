# Volume Price Analysis (VPA)

## What It Is

Volume Price Analysis is the study of the relationship between price movement and trading volume to identify the intent behind market moves. Volume reveals conviction. Price can lie (fake breakouts, stop hunts), but volume is harder to fake at scale.

The core idea: if institutions are buying or selling, volume tells you. Price alone only tells you what happened. Volume tells you how committed the participants were.

---

## Why Volume Matters

- **High volume + price move** = conviction. The move is real.
- **Low volume + price move** = suspicious. Likely a fake-out or exhaustion.
- **High volume + no price move** = absorption. A big player is absorbing selling/buying pressure. Reversal coming.
- **Low volume + no price move** = nobody cares. Avoid.

---

## Types of Volume Analysis

### 1. Vertical Volume (Standard)
The bars you see below your candlestick chart. Measures total volume traded during each time period.

**Key signals:**
- Volume spike on breakout = confirm the breakout
- Volume declining during a trend = trend weakening
- Volume spike at a reversal point = potential capitulation/exhaustion

### 2. Volume Profile (Horizontal Volume)
Shows how much volume was traded at each PRICE level, not time period. Creates a histogram on the side of the chart.

**Key concepts:**
- **Point of Control (POC)**: The price with the most volume. Acts as a magnet -- price tends to return here.
- **Value Area (VA)**: The range containing 70% of volume. Price spends most time here.
- **High Volume Nodes (HVN)**: Price levels with heavy trading. Act as support/resistance.
- **Low Volume Nodes (LVN)**: Price levels with light trading. Price moves through these quickly.

### 3. Volume-at-Price Analysis (VAP)
Similar to volume profile but used by day traders to identify intraday equilibrium levels and where price is likely to stall or accelerate.

---

## Key Volume Indicators

### VWAP (Volume Weighted Average Price)
The true average price of the day, weighted by volume. Resets daily.

**How institutions use it:**
- Institutions benchmark their execution against VWAP
- Buying below VWAP = good fill. Selling above VWAP = good fill.
- Price above VWAP = bullish bias. Price below VWAP = bearish bias.

See the dedicated VWAP strategy file for trading setups.

### On-Balance Volume (OBV)
Cumulative running total: adds volume on up days, subtracts on down days.

**Signals:**
- OBV rising while price is flat = accumulation (bullish)
- OBV falling while price is flat = distribution (bearish)
- OBV divergence from price = early warning of reversal

### Relative Volume (RVOL)
Compares current volume to the average volume for that time of day.

**Interpretation:**
- RVOL > 2x = significant activity, worth watching
- RVOL > 5x = major event, institutional participation likely
- RVOL < 0.5x = dead, avoid trading

---

## Volume Patterns That Matter

### 1. Breakout Confirmation
Price breaks above resistance. Volume should surge.
- High volume breakout = real. Enter on pullback or continuation.
- Low volume breakout = likely fails. Avoid or fade it.

### 2. Climax / Exhaustion Volume
Massive volume spike at the end of a trend. The last buyers are buying at the top (or last sellers at the bottom). Often signals reversal.

**Characteristics:**
- Highest volume bar in the recent trend
- Wide-range candle
- Often accompanied by a long wick (rejection)

### 3. Volume Divergence
Price makes new highs but volume is declining. The trend is losing steam.
- Bearish divergence: higher price, lower volume = weakening uptrend
- Bullish divergence: lower price, lower volume = weakening downtrend

### 4. Absorption / Accumulation
High volume but price barely moves. A large player is absorbing all the selling (or buying) pressure.
- High volume at support with no break = demand absorption = bullish
- High volume at resistance with no break = supply absorption = bearish

### 5. No Supply / No Demand
- **No supply**: Price pulls back on very low volume = sellers are absent. Bullish continuation likely.
- **No demand**: Price rallies on very low volume = buyers are absent. Bearish continuation likely.

---

## How Institutions Leave Footprints

1. **Unusual volume spikes** at specific price levels
2. **Dark pool prints** -- large blocks traded off-exchange
3. **Volume at the open** -- first 30 minutes often reveal institutional bias
4. **Volume profile shifts** -- POC moving higher = accumulation, moving lower = distribution
5. **Block trades** appearing on time & sales

---

## Practical Framework

### Before entering any trade, ask:
1. Is volume confirming the move? (If not, skip it)
2. Where is the volume profile POC? (Price gravitates here)
3. Is RVOL elevated enough to matter? (Need > 1.5x minimum)
4. Are there volume divergences warning of reversal?
5. Is price above or below VWAP? (Determines bias)

---

## ORACLE's Verdict
- **Edge:** Real
- **Difficulty:** Medium
- **Best for:** Day trading / Swing trading (volume profile for intraday, OBV/divergence for swing)
- **Win rate (estimated):** N/A -- volume is a confirmation tool, not a standalone strategy
- **Risk/Reward:** Improves R:R of other strategies when used as a filter
- **Will I use this?** Yes -- as a mandatory confirmation layer
- **Why:** Volume is the one thing that's hardest to fake. Every strategy I use will have a volume confirmation check. VWAP for intraday bias, volume profile for key levels, RVOL for "is this move worth trading," and OBV/divergence for swing trade timing. This isn't a strategy on its own -- it's the filter that separates real moves from noise. Non-negotiable.
