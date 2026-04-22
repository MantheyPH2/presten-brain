# Options Edge Strategies: Real Income Generation

## The Core Principle

Options income strategies work by exploiting one fundamental truth: **options lose value over time (theta decay), and most options expire worthless.** If you're the seller (writer), time is your friend. If you're the buyer, time is your enemy.

Expected returns: 1-3% monthly (12-36% annualized) with 65-85% win rates. This isn't sexy, but it compounds.

---

## Strategy 1: The Wheel (Cash-Secured Puts + Covered Calls)

### How It Works
1. **Phase 1 -- Sell Cash-Secured Puts**: Sell a put option on a stock you WANT to own at a LOWER price. Collect premium.
   - If price stays above your strike: Keep the premium. Repeat.
   - If price drops below your strike: You get assigned (buy) the stock at the strike price (minus premium collected).

2. **Phase 2 -- Sell Covered Calls**: Now that you own the stock, sell call options above your cost basis.
   - If price stays below your strike: Keep the premium. Repeat.
   - If price rises above your strike: Your stock gets called away at the strike price (profit + premium).

3. **Cycle back to Phase 1.** This is the "wheel."

### Real Math Example
- Stock: AAPL at $180
- Sell $175 put, 30 DTE, collect $3.00 premium
- **Scenario A**: AAPL stays above $175. You keep $300. Annualized return: ~24% on capital at risk.
- **Scenario B**: AAPL drops to $170. You buy at $175, but your cost basis is $172 (strike - premium). Now sell covered calls above $172.

### Real Results Data
- $27,000+ profit documented from 160 trades
- Conservative yields: 0.5-1% monthly on deployed capital
- High IV environments: 2-3% monthly
- Low IV environments: 0.5-1% monthly

### Critical Risks
- **Stock drops significantly**: A $5,000 position collecting $150/month in premium loses $1,500 when the underlying drops 30%. That's 10 months of premium wiped out.
- **A 30% drawdown on an individual stock is not a black swan -- it's a Tuesday.**

### Stock Selection Rules (This Is Where People Fail)
- Only wheel on stocks you genuinely want to own long-term
- High-quality, profitable companies with competitive moats
- Avoid high-premium "junk" stocks (high premium = high risk)
- Diversify across 3-5 names minimum
- Never allocate more than 20% of your portfolio to a single wheel position

---

## Strategy 2: Credit Spreads (Bull Put / Bear Call)

### Bull Put Spread (Bullish)
- Sell a put at a higher strike, buy a put at a lower strike (same expiration)
- You collect net credit (income)
- Max profit = net credit received
- Max loss = difference between strikes minus credit
- Win if: price stays above the short put strike

### Bear Call Spread (Bearish)
- Sell a call at a lower strike, buy a call at a higher strike
- Same concept, opposite direction

### Why Credit Spreads Are Better Than Naked Options
- **Defined risk**: You know your max loss before entering
- **Lower capital requirement**: Margin is the spread width, not the full underlying price
- **Same theta decay benefit** as naked options, but with protection

### Ideal Setup
- 30-45 DTE (sweet spot for theta decay)
- Short strike at 1 standard deviation OTM (~84% probability of profit)
- Width: 2-5 points (wider = more credit but more risk)
- Target: Exit at 50% of max profit (don't be greedy)

---

## Strategy 3: Iron Condor

### What It Is
A bull put spread + a bear call spread on the same underlying, same expiration. You profit if price stays within a range.

### When to Use
- Low to moderate implied volatility expected to stay low
- Sideways/range-bound market
- After a large IV spike (expecting IV crush)
- 30-45 DTE

### When to AVOID
- Before major news events
- During trending markets
- When IV is already low (not enough premium to collect)

### Management Rules
- Exit at 50% of max profit
- If the tested side reaches 2x the credit received, close the position
- Or: Roll the untested side closer to collect more credit (advanced)

---

## Strategy 4: Calendar Spreads

### What It Is
- Sell a near-term option, buy a longer-term option at the same strike
- Profit from the near-term option decaying faster than the long-term option

### When to Use
- When you expect price to stay near a specific level
- When IV is low and you expect it to rise (long vega position)
- Around earnings: sell the pre-earnings expiration, buy the post-earnings expiration

### Key Risk
- If price moves significantly away from the strike, both options lose value similarly and the spread collapses

---

## The Math of Consistent Income

### Position Sizing for Options Income
- Never risk more than 5% of your portfolio on a single options position
- Max 20-30% of portfolio allocated to options income strategies
- Diversify across sectors and expiration dates
- Keep 30-40% cash reserve for assignment risk or adjustment capital

### Expected Monthly Income
| Strategy | Monthly Return | Win Rate | Max Loss Per Trade |
|----------|---------------|----------|-------------------|
| Wheel | 1-3% | 70-80% | Significant (stock drop) |
| Credit Spreads | 0.5-2% | 75-85% | Spread width - credit |
| Iron Condors | 1-3% | 70-80% | Spread width - credit |
| Calendars | Variable | 55-65% | Debit paid |

---

## ORACLE's Verdict
- **Edge:** Real
- **Difficulty:** Medium
- **Best for:** Income generation / Portfolio enhancement
- **Win rate (estimated):** 70-85% (credit strategies)
- **Risk/Reward:** Asymmetric against you (small frequent wins, occasional large losses)
- **Will I use this?** Yes -- credit spreads and selective wheel strategy
- **Why:** Options income strategies provide a genuine, mathematically sound edge because of theta decay. The wheel is excellent for building positions in stocks you want to own anyway. Credit spreads provide defined-risk income with high win rates. The critical insight is that the high win rate is balanced by occasional large losses -- you MUST manage losers quickly and size positions conservatively. I'll use credit spreads as my primary income strategy (defined risk), the wheel on 2-3 high-quality stocks, and iron condors in range-bound markets. The 50% profit exit rule is non-negotiable -- greed kills options sellers.
