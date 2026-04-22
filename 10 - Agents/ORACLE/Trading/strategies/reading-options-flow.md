# Reading Options Flow & Dark Pool Data

## What It Is

Options flow analysis tracks large, unusual options trades to identify what institutional and informed traders are betting on. Dark pool data reveals large block trades executed off-exchange where institutions operate to minimize market impact.

This is essentially "following the smart money" through their options and off-exchange activity.

---

## Unusual Options Activity (UOA)

### What Qualifies as "Unusual"
- Volume 5x+ higher than the average daily volume for that option
- Volume significantly exceeding open interest (new positions being opened)
- Large premium traded ($500K+ on a single trade)
- Trades executed above the ask price (buyer was urgent -- paid premium for speed)

### How to Read UOA Signals

**Strong Bullish Signals:**
- Large call buying above the ask (sweeps)
- Repeated call buying across multiple strikes/expirations
- Put selling (writing) at scale
- Volume >> open interest on calls

**Strong Bearish Signals:**
- Large put buying above the ask
- Repeated put buying across multiple strikes
- Call selling at scale
- Volume >> open interest on puts

### Key Metrics to Check
1. **Volume-to-Open Interest Ratio**: Volume >> OI = new positions. Strong signal.
2. **Open Interest Changes**: If OI increases the next day, positions were held overnight (conviction). If OI doesn't increase, it was just intraday trading (less meaningful).
3. **Trade Price vs Bid/Ask**: At/above ask = buyer initiated (bullish). At/below bid = seller initiated (bearish).
4. **Repeated Flow**: Multiple large trades in the same direction over hours/days. Much stronger than a single large trade.

---

## Sweep Orders (The Big Signal)

A sweep order is a large order that is simultaneously routed to multiple exchanges to get filled as fast as possible. The buyer literally sweeps all available liquidity at multiple price levels.

**Why sweeps matter**: The trader is so urgent they're willing to pay increasingly higher prices just to get filled NOW. This signals:
- Time-sensitive information
- High conviction
- Institutional size

**How to identify:**
- Marked as "sweep" on options flow platforms
- Fills at multiple exchanges simultaneously
- Often trades above the ask price
- Large dollar amounts ($250K+)

---

## Dark Pool Data

### What Dark Pools Are
Alternative trading systems where institutional investors execute large block trades without showing their orders on public exchanges. About 40% of all US equity volume now trades in dark pools.

### Key Dark Pool Signals

**Block Trades**: Single transactions > 10,000 shares or $200,000.

**Accumulation Signals (Bullish):**
- Sustained dark pool buying while stock trades sideways or dips
- Increasing dark pool volume relative to 30-day average (500%+ surge)
- Price "anchoring" at a specific level where large dark pool trades execute

**Distribution Signals (Bearish):**
- Heavy dark pool selling while stock is still elevated
- Dark pool sell prints clustering at a specific price level
- Divergence: dark pool selling + surface-level bullish price action

### Reading Dark Pool Prints
- One single dark pool print is rarely significant
- **Clusters** of dark pool prints at similar prices = institutional accumulation/distribution
- Dark pool prints far from the current price = potentially pre-positioned for a catalyst
- Compare dark pool volume to normal volume. Higher ratio = more institutional activity.

---

## Combining Flow + Dark Pool for Trade Ideas

### The High-Conviction Setup
1. Unusual call buying (sweeps above ask, high vol/OI ratio)
2. Dark pool accumulation at current price levels
3. Technical setup supports the direction (pullback to support, breakout formation)
4. No conflicting signals (no large put buying, no dark pool selling)

### Red Flags (Noise, Not Signal)
- Single large trade with no follow-through
- Options activity around earnings (often just hedging)
- Activity in low-liquidity options (wide spreads = unreliable)
- Trades at the bid (could be selling to close, not new bearish positions)

---

## Tools for Options Flow & Dark Pool Data

### Free/Cheap
- Barchart.com (unusual options activity screener)
- Yahoo Finance (options chains with volume data)
- CBOE (market data)

### Paid (Worth It)
- **OptionStrat Flow** -- real-time unusual activity, sweep detection
- **InsiderFinance** -- options flow + dark pool integration
- **BlackBoxStocks** -- dark pool scanning + options flow
- **Unusual Whales** -- popular options flow platform
- **FlowAlgo** -- institutional flow tracking
- **Bookmap** -- order flow visualization

---

## How to Filter Signal from Noise

### Signal (Act On)
- Repeated flow in one direction over multiple days
- Large sweep orders ($500K+) in OTM options with 30-60 DTE
- Dark pool accumulation contradicting surface price action
- Flow + technical confluence (bullish flow at a support level)

### Noise (Ignore)
- Single large trade (could be hedging)
- Activity right before/after earnings (usually hedging)
- Penny stock options flow (manipulation risk)
- Flow without volume confirmation on the underlying

---

## ORACLE's Verdict
- **Edge:** Real (when filtered properly)
- **Difficulty:** Hard
- **Best for:** Swing trading / Directional options trades
- **Win rate (estimated):** 55-65% (when properly filtered with technical confirmation)
- **Risk/Reward:** 1:2 to 1:5 (options trades based on flow)
- **Will I use this?** Yes -- as a trade idea generator and conviction builder
- **Why:** Options flow and dark pool data provide something most retail analysis lacks: visibility into what big money is actually doing. Sweep orders and sustained dark pool accumulation/distribution are genuine signals. The challenge is filtering signal from noise -- most individual flow prints are meaningless, but patterns of flow over days, combined with technical setups, create high-conviction trade ideas. I'll use flow data as a "second opinion" and conviction builder, never as a standalone entry signal. The key insight: if smart money is betting the same direction as my technical analysis, that's confluence worth acting on.
