# Options Greeks in Practice (Not Textbook Definitions)

## The Core Trade-Off You Must Understand

Every options position is fundamentally a bet: **Will the realized movement (gamma) exceed the cost of time (theta)?**

- **Long options**: You pay theta every day but benefit from gamma (accelerating gains when price moves your way)
- **Short options**: You collect theta every day but suffer from gamma (accelerating losses when price moves against you)

That's it. Every options strategy is some combination of this trade-off.

---

## Delta: Your Directional Exposure

### What It Actually Tells You
- How much your option's price changes per $1 move in the stock
- **Also doubles as a rough probability estimate** of expiring in-the-money
- A 0.30 delta call has roughly a 30% chance of expiring ITM

### Practical Decision-Making

**Choosing strikes based on delta:**
| Delta | Use Case |
|-------|----------|
| 0.80-0.90 | Deep ITM. Stock replacement. Minimal time premium. |
| 0.50 | ATM. Maximum time value. Balanced risk. |
| 0.30 | OTM. Cheaper. Lower probability but higher leverage. |
| 0.15-0.20 | Far OTM. Lottery ticket. Premium selling sweet spot. |
| 0.05-0.10 | Very far OTM. Almost certainly expires worthless. Wing protection. |

**For income strategies (selling)**: Sell at 0.15-0.30 delta (70-85% probability of keeping premium).

**For directional bets (buying)**: Buy at 0.40-0.60 delta (enough skin in the game, not overpaying for time value).

**Portfolio delta**: Sum up the delta of all your positions. That's your net directional exposure in equivalent shares. If you're delta +500, you're effectively long 500 shares. Manage this actively.

---

## Gamma: The Accelerator (And Your Biggest Risk)

### What It Actually Tells You
- How fast your delta changes as price moves
- Gamma is highest for ATM options near expiration

### Practical Decision-Making

**Why gamma matters for buyers:**
- When you're right, delta increases in your favor (profits accelerate)
- When you're wrong, delta decreases (losses decelerate)
- This is why long options have a "convex" payoff -- limited downside, accelerating upside

**Why gamma is DANGEROUS for sellers:**
- When price moves against you, your loss accelerates
- Near expiration, gamma explodes -- a small price move can create massive delta changes
- **The "gamma risk" near expiration**: A short ATM option with 1 DTE can go from breakeven to max loss in an hour

**Practical rules:**
- If you're short options, avoid holding through the last week before expiration (gamma risk skyrockets)
- Close short options at 50% profit and don't hold for the last drop of theta
- If you're long options, ATM options near expiration give maximum gamma (but you need the move to happen fast)

---

## Theta: The Daily Tax (Or Daily Income)

### What It Actually Tells You
- How much value your option loses each day, all else being equal
- **Theta accelerates as expiration approaches**: Options lose value slowly 60+ days out, then rapidly in the final 30 days. The last week sees 3-5x faster decay than at 60 days.

### Practical Decision-Making

**For options sellers (theta is your friend):**
- Open positions at 30-45 DTE (best theta/gamma ratio)
- Close at 50% profit or at 21 DTE (whichever comes first)
- This captures the best part of the theta curve while avoiding the gamma risk of the final week

**For options buyers (theta is your enemy):**
- Buy with 60+ DTE to minimize daily decay
- Avoid holding below 21 DTE unless you're in profit and riding the move
- Front-month options are cheap but decay fast. Back-month options cost more but give you time to be right.

**The 45 DTE sweet spot**: At 45 days, you capture ~1/3 of the total theta in just the first half of the option's life, with much less gamma risk than the final days. This is why most professional options sellers target this window.

---

## Vega: Your Volatility Bet

### What It Actually Tells You
- How much your option's price changes when implied volatility moves by 1 percentage point
- All options have positive vega (they gain value when IV rises)

### Practical Decision-Making

**Before buying options, check the IV rank/percentile:**
- IV Rank > 50%: IV is elevated. Options are expensive. Favor selling strategies.
- IV Rank < 30%: IV is low. Options are cheap. Favor buying strategies.
- IV Rank 30-50%: Neutral. Either approach works.

**Earnings and vega:**
- Pre-earnings: IV is elevated (high vega exposure on long options)
- Post-earnings: IV crushes (vega collapses)
- If you're long options through earnings, vega crush will hurt you even if you're directionally right

**VIX as a guide:**
- VIX > 25: High IV environment. Sell premium. Your vega exposure on short options will profit as VIX normalizes.
- VIX < 15: Low IV environment. Buy premium. Cheap options with room for IV expansion.

---

## Portfolio-Level Greek Management

### Before Every New Trade, Ask:

1. **What does this do to my portfolio delta?** (Am I getting too directional?)
2. **What's my portfolio theta?** (Positive = I'm collecting. Negative = I'm paying.)
3. **What's my portfolio vega?** (Long vega = I benefit from vol rise. Short vega = I benefit from vol drop.)
4. **What's my gamma exposure?** (Am I vulnerable to a sharp move?)

### Ideal Portfolio Greek Profile for Income Trading
- Delta: Near neutral (slightly long if bullish outlook)
- Theta: Positive (collecting daily)
- Vega: Slightly negative (benefiting from IV normalization)
- Gamma: Slightly negative (the cost of positive theta)

### When to Adjust
- Portfolio delta exceeds +/- 200 shares equivalent: Reduce or hedge
- Theta is negative and you're not in a directional trade: Something's wrong
- Vega exposure is too large before a vol event: Reduce position size or hedge

---

## ORACLE's Verdict
- **Edge:** Real (understanding Greeks is essential for any options trader)
- **Difficulty:** Medium
- **Best for:** All options trading styles
- **Win rate (estimated):** N/A -- Greeks are tools, not strategies
- **Risk/Reward:** Improves both by enabling better decision-making
- **Will I use this?** Yes -- mandatory for every options trade
- **Why:** The Greeks transform options trading from gambling into informed decision-making. The theta/gamma trade-off is the most important concept: short premium collects theta but risks gamma, long premium pays theta but benefits from gamma. The 30-45 DTE entry, 50% profit exit framework for selling, and the 60+ DTE purchase window for buying, are backed by decades of professional options trading data. Portfolio-level Greek management prevents the "I have 10 positions that all lose at the same time" disaster. Non-negotiable knowledge for any options trader.
