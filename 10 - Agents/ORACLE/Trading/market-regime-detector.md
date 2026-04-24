# Market Regime Detector

**Classify the regime before every trade. The strategy must match the regime. A good strategy in the wrong regime is still a losing strategy.**

Last classified: {{date}}
Current regime: ______

---

## The Core Principle

Markets cycle through regimes. No single strategy works in all regimes. The traders who lose money are not always using bad strategies — they are using good strategies in the wrong environment. An iron condor in a trending market is not a neutral strategy. It is a slow bleed.

ORACLE's job is to identify the regime first, then select the strategy. Never the reverse.

---

## Regime 1: Trending

### What It Looks Like

Price is moving directionally with conviction. Higher highs and higher lows in an uptrend. Lower highs and lower lows in a downtrend. Pullbacks are shallow and resume the primary direction. The market has a destination.

### Signals (need at least 4 of 6 to confirm)

| Signal | Bullish Trend | Bearish Trend | Current |
|--------|--------------|--------------|---------|
| ADX | Above 25 | Above 25 | |
| Price vs 20-day SMA | Above | Below | |
| Price vs 50-day SMA | Above | Below | |
| Price structure | Higher highs / higher lows | Lower highs / lower lows | |
| Market breadth | More advancing than declining | More declining than advancing | |
| Volume on trend days | Higher on up days | Higher on down days | |

**Confirmed trending: YES / NO**

### Strategies to Use

- **Debit spreads in the trend direction** — buy the move, define risk
- **LEAPS** — long-dated calls (uptrend) or puts (downtrend) to capture multi-week moves
- **Momentum entries** — buy breakouts from consolidation, enter pullbacks to the 20-day SMA
- **Diagonal spreads** — collect premium on short leg while holding long LEAP
- **Call spreads (uptrend) / Put spreads (downtrend)** — defined risk with high probability of profit in strong trends

### Strategies to Avoid

- Iron condors — the short strikes will be threatened by directional movement
- Naked short strangles — unlimited risk on one side as the trend accelerates
- Selling premium against the trend (e.g., selling calls in a strong uptrend)
- Fading the trend without a specific reversal catalyst

### Position Sizing in a Trend

- Full size is acceptable: up to 2% risk per trade
- Can run 3-4 positions in trend direction if well-staggered in time and instrument
- Do not pyramid aggressively — add on proven pullbacks, not new highs

### How to Detect the Shift Before It Happens

Early warning signs a trend is ending:
1. ADX begins declining from above 30 (trend losing momentum, not necessarily reversing yet)
2. Price fails to make a new high (uptrend) or new low (downtrend) on 2+ consecutive attempts
3. Volume dries up on trend continuation days
4. Breadth divergence: index makes new high but fewer stocks participating
5. VIX stops falling and starts rising during an uptrend
6. Key moving average (20-day) flattens or reverses slope
7. Price closes below the 20-day SMA for the first time in weeks

**If 3+ early warning signs appear: move to Transition protocol immediately.**

---

## Regime 2: Range-Bound

### What It Looks Like

Price oscillates between a defined support level and a defined resistance level. There is no net directional progress over days or weeks. Breakout attempts fail and price reverts to the middle. This is the premium seller's environment.

### Signals (need at least 4 of 6 to confirm)

| Signal | Value | Confirms Range? | Current |
|--------|-------|-----------------|---------|
| ADX | Below 20 | YES | |
| VIX | Below 18 | YES | |
| Price action | Identifiable support and resistance | YES | |
| Failed breakouts | 1+ in recent weeks | YES | |
| 20-day SMA slope | Flat | YES | |
| Bollinger Band width | Contracting | YES | |

**Confirmed range-bound: YES / NO**

### Support level: $ ______
### Resistance level: $ ______
### Range width: $ ______ / ______ %

### Strategies to Use

- **Iron condors** — sell both sides, collect premium, profit from staying inside the range
- **Credit spreads** — sell the strike near resistance (call spread) or near support (put spread)
- **The Wheel** — sell cash-secured puts at or below support; if assigned, sell covered calls at or above resistance
- **Calendar spreads** — exploit low IV and time decay in a stuck market
- **Covered calls** — generate income on existing equity positions going nowhere
- **Short strangles (sized appropriately)** — collect on both sides of the range

### Strategies to Avoid

- Long calls or puts without defined risk (directional positions bleed theta in a range)
- Chasing breakouts without confirmation — they statistically fail in range regimes
- LEAPS in the direction of a hoped-for breakout — time decay destroys value while waiting

### Position Sizing in a Range

- Full size is acceptable for income strategies: up to 2% risk per trade
- Iron condors and credit spreads can be stacked across multiple instruments in the same regime
- Tighten stops if price approaches the edges of the range — these are the decision points

### How to Detect the Shift Before It Happens

Early warning signs a range is ending (breakout coming):
1. ADX rises above 20 from below — trend energy beginning to build
2. VIX expands from low levels — market preparing for a move
3. Price makes a clean close above resistance or below support with volume
4. Bollinger Bands begin expanding after a contraction
5. Increasing volume on the days that test the range boundaries
6. News catalyst or scheduled event approaching (earnings, FOMC, major data release)
7. Put/call ratio shifts sharply in one direction

**If a range break occurs with volume confirmation: the range is over. Close income positions and evaluate for trend strategies.**

---

## Regime 3: Volatile

### What It Looks Like

Large daily moves in both directions. Gap opens that do not fill. News-driven price action. High VIX. No predictable structure — support and resistance levels are sliced through without holding. This is not a trading environment; it is a survival environment.

### Signals (need at least 3 of 5 to confirm)

| Signal | Threshold | Current |
|--------|-----------|---------|
| VIX level | Above 25 | |
| Average daily range (ADR) | 2x normal | |
| Gap opens | 3+ in past 2 weeks | |
| Price vs recent support | Support failing regularly | |
| News environment | High-frequency macro events | |

**Confirmed volatile: YES / NO**

### Strategies to Use

- **Protective puts** — hedge existing long positions immediately if not already hedged
- **Long straddles** — buy both a call and a put on a major index or volatile underlying; profit from a large move in either direction
- **Long strangles** — cheaper than straddles, requires an even larger move; use when IV has not yet spiked but you expect it to
- **VIX calls** — direct volatility exposure without needing to pick direction
- **Reduced-size directional trades** — if you must trade direction, cut normal position size by 50%
- **Cash** — doing nothing is a valid and often optimal strategy in a volatile regime

### Strategies to Avoid

- Selling premium (credit spreads, iron condors, covered calls) — you will be selling cheap options into a vol spike and then watching them explode
- Full-size positions in any direction — volatility means wide stops and large losses
- Adding to losing positions — volatile regimes accelerate losses
- Catching falling knives — reversal trades before the regime stabilizes

### Position Sizing in Volatile Regimes

- Maximum position size is 50% of normal (1% max risk per trade instead of 2%)
- Portfolio heat cap drops to 3% total (half the normal 6%)
- If VIX exceeds 35: no new positions, only defensive actions on existing ones
- If VIX exceeds 40: close all speculative positions, hold only defined-risk positions

### How to Detect the Shift Before It Happens

Early warning signs volatility regime is beginning:
1. VIX breaks above 20 from a period of sub-18 readings
2. VIX futures curve inverts (near-term VIX above longer-dated VIX)
3. Credit spreads in high-yield bonds begin widening
4. Safe haven buying appears (TLT, GLD, JPY, CHF rallying)
5. Increasing gap-open frequency on SPY
6. Major macro event approaching with uncertain outcome (unscheduled Fed action, geopolitical escalation, financial institution stress)
7. Put/call ratio spikes above 1.2

**If 3+ early warning signs appear: immediately hedge, reduce position sizes, raise cash.**

---

## Regime 4: Transition

### What It Looks Like

The regime is changing but has not yet established a new state. Indicators are conflicting. Breakouts fail. Trend setups get stopped out. Income strategies get tested at the edges. The market does not know what it wants to do. This is the most dangerous regime because it generates false signals that look like real ones.

### Signals (any 2 indicate transition)

- ADX between 20 and 25 (borderline — not clearly trending or range-bound)
- Recent regime signals failing (trend trades reversing, range trades breaking out)
- VIX expanding from a low base (leaving range-bound, entering volatile)
- Price making choppy, overlapping candles on the daily chart
- Key moving averages crossing or price whipsawing around them
- Volume declining on trend days, no conviction in either direction

### What to Do in a Transition

1. **Immediately cut all position sizes by 50%.** Half size in a transition, always.
2. **Tighten all stop losses.** Give positions less room, not more.
3. **Add hedges** — this is the time to buy protection before it becomes expensive.
4. **Close any positions with wide stops or high risk.** Transitions produce sudden moves.
5. **Do not open new positions** unless they are:
   - Defined risk only (spreads, not naked positions)
   - Half normal size
   - With a clear regime resolution trigger (e.g., "I will only hold this if price closes above X")
6. **Wait for confirmation.** The transition will resolve into one of the three primary regimes. Let it resolve before trading full size again.

### Why Transition is the Most Dangerous

In a clear trend, being wrong costs you your stop. In a clear range, being wrong costs you your stop. In a transition, your stop gets hit, you re-enter, your stop gets hit again, you re-enter again, and three losses later you have taken your maximum weekly loss without ever being catastrophically wrong on a single trade. The market chops you to pieces with small cuts.

The solution is to reduce size dramatically and accept that the best trade in a transition is often no trade at all.

---

## Regime Classification Worksheet

Use this worksheet at the start of each week and whenever you suspect a regime change.

### Step 1: Measure the signals

| Indicator | Current Value | Implication |
|-----------|--------------|-------------|
| ADX (14-period) | | <20=range, 20-25=unclear, >25=trend |
| VIX spot | | <18=calm, 18-25=normal, >25=volatile |
| SPY vs 20-day SMA | | above=bullish, below=bearish |
| SPY vs 50-day SMA | | above=bullish, below=bearish |
| SPY daily structure | | HH/HL=uptrend, LH/LL=downtrend, choppy=range/transition |
| Bollinger Band width | | expanding=vol increasing, contracting=range |
| VIX futures curve | | normal=contango, stressed=backwardation |

### Step 2: Count the votes

- Trending signals: ______
- Range-bound signals: ______
- Volatile signals: ______
- Conflicting signals: ______

### Step 3: Classify

- Clear majority → assign that regime
- Mixed signals → Transition protocol
- When in doubt → assume Transition and size down

### Current classification: ______

### Confidence in classification: HIGH / MEDIUM / LOW

### Next review trigger: ______

---

## Regime History Log

Track regime changes over time to understand cycle length and your classification accuracy.

| Date | Regime | Confidence | Changed From | Correct? |
|------|--------|-----------|-------------|---------|
| | | | | |
| | | | | |
| | | | | |

---

## Quick Reference Card

| Regime | ADX | VIX | Strategy | Max Size | Watch For |
|--------|-----|-----|----------|----------|-----------|
| Trending | >25 | Any | Debit spreads, LEAPS, momentum | 2% | ADX declining, breadth divergence |
| Range-bound | <20 | <18 | Iron condors, credit spreads, Wheel | 2% | VIX expansion, range break on volume |
| Volatile | Any | >25 | Straddles, protective puts, cash | 1% | VIX reverting, structure returning |
| Transition | 20-25 | Rising | Reduce, hedge, wait | 1% | Clear regime establishing |

---

*The market does not owe you a clear regime. Sometimes it is genuinely ambiguous. In those moments, the correct position size is smaller than you think, and the correct number of positions is fewer than you want.*
