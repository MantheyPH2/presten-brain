# Options Mastery Path

> Not theory. Real edge. Every level builds on the last. Do not skip levels. Paper trade until you stop making the common mistakes listed, then move on.

---

## Level 1: Foundation (Week 1-2)

### What You Need to Know

**Options Pricing Mechanics (Black-Scholes Intuition)**
Forget the math. Here is what matters: an option's price is driven by five things:
1. **Intrinsic value** — how far in-the-money the option is (stock price minus strike for calls)
2. **Time value** — more time = more expensive, decays non-linearly (accelerates in final 30 days)
3. **Implied volatility** — the market's forecast of how much the stock will move. Higher IV = more expensive options. This is the single most important variable.
4. **Interest rates** — minor effect, ignore for now
5. **Dividends** — only matters for deep ITM calls near ex-div dates

The key insight: you are not just betting on direction. You are betting on direction AND timing AND volatility. Most beginners only think about #1.

**The Greeks in Real Trading**

| Greek | What It Actually Means | How It Hits Your P&L |
|-------|----------------------|---------------------|
| **Delta** | How much the option moves per $1 stock move. A 0.50 delta call gains $50 per $1 up. Also approximates probability of expiring ITM. | You check delta first. It tells you your directional exposure RIGHT NOW. 10 contracts at 0.30 delta = 300 shares equivalent. |
| **Gamma** | How fast delta changes. High gamma = delta shifts rapidly. Near-expiration ATM options have extreme gamma. | Gamma is why 0DTE options are thrilling and deadly. Your position can go from 0.30 delta to 0.80 delta in minutes. You think you have a small position, then suddenly you don't. |
| **Theta** | How much the option loses per day from time decay. A theta of -0.05 means you lose $5/contract/day. | Theta is your friend when selling, your enemy when buying. It accelerates: a 30-DTE option loses ~3% of value/day, a 7-DTE option loses ~10%/day. |
| **Vega** | How much the option price changes per 1% move in IV. Vega of 0.10 means option gains $10 per 1% IV increase. | Vega is why your calls lost money even though the stock went up. IV dropped 5%, vega was 0.15, you lost $75/contract from vol alone while gaining $50 from delta. Net loss. |

**IV Rank vs IV Percentile**

- **IV Rank**: where current IV sits relative to the past year's range. IV is 25, 52-week low is 15, high is 45. IV Rank = (25-15)/(45-15) = 33%. Simple but skewed by outlier spikes.
- **IV Percentile**: what percentage of days in the past year had lower IV than today. More reliable.
- **When options are cheap**: IV Rank < 20 or IV Percentile < 25. Buy strategies (debit spreads, long straddles, LEAPS).
- **When options are expensive**: IV Rank > 50 or IV Percentile > 60. Sell strategies (credit spreads, iron condors, covered calls).
- Check IV rank on every single trade. It is the most neglected edge in retail options trading.

**Reading an Options Chain**

What to look for, in order:
1. **Bid-ask spread** — tight spreads (pennies) mean liquid. Wide spreads ($0.50+) mean you are giving away edge on entry. Never trade illiquid options.
2. **Open interest** — >500 OI for any strike you trade. Below that, you will have trouble getting fills and exiting.
3. **Volume** — unusual volume vs OI ratio signals institutional activity. Volume 5x OI = someone knows something.
4. **Put/call volume ratio** — heavy put buying = hedging or bearish bets. Heavy call buying = bullish.
5. **IV skew** — if OTM puts are much more expensive (higher IV) than OTM calls, the market is pricing downside risk. Normal for indices, meaningful for individual stocks.
6. **Expected move** — ATM straddle price approximates the market's expected move for that expiration.

**Basic Strategies**

| Strategy | When | Max Gain | Max Loss | Break-Even |
|----------|------|----------|----------|------------|
| Long call | Bullish, expect move up soon | Unlimited | Premium paid | Strike + premium |
| Long put | Bearish, expect move down soon | Strike - premium | Premium paid | Strike - premium |
| Covered call | Own shares, mildly bullish, want income | Premium + (strike - stock price) | Stock drops to zero minus premium received | Stock price - premium |
| Cash-secured put | Want to buy stock cheaper, bullish | Premium received | Strike - premium (assigned and stock drops) | Strike - premium |

### What to Study

- **YouTube**: tastylive "Options Crash Course" series (Mike Butler), projectfinance "Options Trading for Beginners" playlist, InTheMoney's Greek explanations
- **Books**: "Options as a Strategic Investment" by McMillan (reference, don't read cover to cover), "The Options Playbook" by Brian Overby (practical starter)
- **Tools**: Set up a free ThinkorSwim paper trading account. Use the Analyze tab to visualize payoff diagrams. Use the "thinkBack" tool to see historical options pricing.

### What to Practice

Paper trade 10 basic options trades:
- 3 long calls on stocks you are bullish on (30-45 DTE, ATM or slightly OTM)
- 3 long puts on stocks you are bearish on (30-45 DTE, ATM or slightly OTM)
- 2 covered calls on stocks you "own" in paper (30 DTE, ~0.30 delta OTM)
- 2 cash-secured puts on stocks you want to buy (30 DTE, at a strike you'd be happy owning)

Log every trade: entry price, Greeks at entry, IV rank, thesis, result, what you learned.

### How to Know You're Ready for Level 2

- You can explain delta, gamma, theta, vega in one sentence each without looking anything up
- You understand why a call can lose money even when the stock goes up (vega/theta)
- You check IV rank before every trade automatically
- You never trade options with bid-ask spreads wider than 10% of the mid price
- You completed 10 paper trades and journaled them all

### Common Mistakes at This Level

1. **Buying OTM calls because they're cheap** — they are cheap because they almost always expire worthless. The probability of profit is 15-25%.
2. **Ignoring IV** — buying calls when IV rank is 80 means you overpaid. The stock has to move MORE than expected just to break even.
3. **Holding until expiration** — most options should be closed at 50% profit or 21 DTE, whichever comes first. Time decay accelerates and gamma risk increases.
4. **Trading illiquid options** — that $0.20 bid-ask spread on a $1.00 option means you lose 10% on entry and 10% on exit.
5. **Position too large** — no single options trade should risk more than 2-3% of your portfolio.

---

## Level 2: Income Strategies (Week 3-4)

### What You Need to Know

**The Wheel Strategy Deep Dive**

The Wheel is the most reliable options income strategy for stocks you want to own. Here is the cycle:

1. **Sell cash-secured put (CSP)** at a strike you'd be happy buying the stock
   - Pick a strike at ~0.30 delta (70% chance of expiring OTM)
   - Collect premium. If it expires OTM, keep premium, sell another CSP.
   - 30-45 DTE, close at 50% profit or 21 DTE

2. **If assigned** — you now own 100 shares at the strike price minus premium collected. Your cost basis is lower than the market was when you sold the put.

3. **Sell covered call (CC)** at or above your cost basis
   - Pick a strike at ~0.30 delta
   - Collect premium. If it expires OTM, keep premium, sell another CC.
   - 30-45 DTE

4. **If called away** — you sold shares at the strike price plus you kept all the premium. Return to step 1.

**Best Wheel candidates:**
- Stocks you genuinely want to own long term (not meme stocks)
- IV rank > 30 (enough premium to be worth it)
- Stock price that allows 100 shares without concentrating portfolio (typically $20-$100 range)
- Liquid options (tight bid-ask, high OI)
- Examples: AAPL, AMD, SOFI, PLTR, F, BAC

**Credit Spreads — The Bread and Butter**

Credit spreads are the core income strategy. You sell an option and buy a cheaper one further OTM for protection.

*Bull Put Spread (bullish):*
- Sell a put at strike A, buy a put at lower strike B
- Max profit = net credit received
- Max loss = width of spread minus credit received
- Example: stock at $100. Sell $95 put for $2.50, buy $90 put for $1.00. Net credit $1.50. Max loss = $5 - $1.50 = $3.50. Risk/reward: risk $3.50 to make $1.50 (43% return if it works, and it works ~70% of the time at 0.30 delta short strike).

*Bear Call Spread (bearish):*
- Sell a call at strike A, buy a call at higher strike B
- Same mechanics, opposite direction

**Key rules for credit spreads:**
- 30-45 DTE, 0.30 delta short strike
- Width of 3-5 strikes ($3-$5 wide for most stocks)
- Close at 50% of max profit — do NOT get greedy waiting for full credit
- Close at 2x the credit received as a stop loss (took in $1.50, close if loss hits $3.00)
- Never risk more than 2-3% of portfolio per spread

**Iron Condors — Range-Bound Markets**

An iron condor = bull put spread + bear call spread on the same stock/expiration.

- Sell the $95/$90 put spread AND the $105/$110 call spread
- Collect credit from both sides
- Profit zone: stock stays between $95 and $105
- Works best when: IV rank > 50 (expensive premiums), stock is range-bound, no catalysts expected
- Close at 50% of max profit
- Adjust or close if stock breaches a short strike

**Calendar Spreads — Playing Time Decay**

- Sell a near-term option, buy a longer-term option at the same strike
- You profit from the near-term option decaying faster (theta differential)
- Works best: IV rank is low on the front month, you expect the stock to stay near the strike
- Example: stock at $100. Sell the 30-DTE $100 call for $3.00, buy the 60-DTE $100 call for $5.00. Net debit $2.00. If stock stays near $100, the front month decays faster, spread widens, profit.
- Risk: stock moves far from the strike in either direction

**Ratio Spreads — Advanced Income**

- Buy 1 ATM call, sell 2 OTM calls (1x2 ratio)
- Can be done for a small debit or even a credit
- Profits if stock goes up moderately but not too much
- Unlimited risk above the upper strike if done naked — use a further OTM long call to cap it (turning it into a broken wing butterfly)
- Only use ratio spreads when you have Level 3+ understanding

### What to Study

- **YouTube**: tastylive "Strategies" playlist (they are the credit spread masters), Options Alpha "Wheel Strategy" series, SMB Capital options videos
- **Books**: "Trading Options Greeks" by Passarelli (how Greeks interact in real spreads)
- **Tools**: Use ThinkorSwim's spread order entry. Practice the roll mechanics on paper.

### What to Practice

Run the Wheel on 2 stocks for 2 weeks in paper trading:
- Pick 2 stocks with IV rank > 30, liquid options, price $20-$80
- Sell CSPs at 0.30 delta, 30-45 DTE
- If assigned, immediately sell CCs
- Track all premium collected, cost basis, assignment events

Also paper trade:
- 5 credit spreads (mix of bull puts and bear calls)
- 2 iron condors on range-bound stocks or ETFs (SPY or IWM)
- 1 calendar spread

### How to Know You're Ready for Level 3

- You can set up any credit spread in under 60 seconds on your platform
- You understand probability of profit vs risk/reward tradeoff
- You close winners at 50% and losers at 2x mechanically, without emotion
- You have run 2 full Wheel cycles in paper trading
- You understand the difference between selling premium at high IV vs low IV

### Common Mistakes at This Level

1. **Not closing at 50% profit** — waiting for full max profit means you are in the trade for the last 50% of profit but taking on 80% of the remaining risk. Close at 50%.
2. **Selling credit spreads in low IV** — the premium is not worth the risk. IV rank should be > 30 for selling strategies.
3. **Iron condors on trending stocks** — iron condors work in range-bound markets. If a stock is trending, one side will blow up.
4. **Wheeling bad stocks** — never Wheel a stock you don't want to own. NKLA at $8 has high IV for a reason.
5. **Too many positions** — start with 3-5 positions max. You cannot monitor 20 credit spreads effectively.

---

## Level 3: Directional Edge (Week 5-6)

### What You Need to Know

**Debit Spreads for Directional Bets**

When you have a directional thesis and IV is low (options are cheap), debit spreads are superior to naked long options:
- Cheaper than buying a single option (lower max loss)
- Theta works against you less aggressively
- Defined risk, defined reward

*Bull Call Spread:*
- Buy ATM or slightly ITM call, sell OTM call
- Example: stock at $100, bullish to $110. Buy $100 call for $5.00, sell $110 call for $2.00. Net debit $3.00. Max profit $7.00 (strike width minus debit). Risk/reward: risk $3 to make $7.
- Choose 45-60 DTE to give the thesis time to play out
- Close at 50-75% of max profit

**LEAPS as Stock Replacement**

A LEAPS (Long-term Equity Anticipation Security) is simply a long-dated option, typically 1-2 years out.

- Buy a deep ITM LEAPS call (delta 0.80+, typically 20-30% ITM)
- Costs ~20% of the stock price but captures ~80% of the upside
- Example: AAPL at $200. Buy the $160 call expiring in 18 months for $48. That is $4,800 vs $20,000 for 100 shares. If AAPL goes to $220, the LEAPS gains ~$16 ($1,600) vs shares gaining $2,000. You captured 80% of the move with 24% of the capital.
- Risks: stock drops significantly and your LEAPS loses most of its value (but max loss is the premium). Time decay is minimal on 18-month options ($0.03-0.05/day theta).
- When to use: long-term conviction in a stock, want capital efficiency, willing to accept the LEAPS could go to zero in a crash

**Poor Man's Covered Call (PMCC)**

Combine LEAPS with short-term call selling:
1. Buy deep ITM LEAPS (0.80+ delta, 12-18 months out)
2. Sell monthly OTM calls against it (0.20-0.30 delta, 30-45 DTE)
3. Collect theta income just like a covered call, but with 75-80% less capital

- Example: instead of buying 100 shares of MSFT at $400 ($40,000), buy the $320 LEAPS call for $90 ($9,000). Sell monthly $420 calls for $4 ($400/month). Annualized: ~$4,800 income on $9,000 capital = 53% annualized yield on capital deployed.
- Risk: stock drops below LEAPS strike and you lose most of the LEAPS value. Stock rockets past your short call strike and your gains are capped.
- Management: roll the short call up and out if it gets challenged. Roll the LEAPS if it drops below 0.60 delta.

**Butterfly Spreads — Cheap Lottery Tickets**

A butterfly = buy 1 lower strike, sell 2 middle strike, buy 1 upper strike (all same expiration).
- Max profit at the middle strike at expiration
- Very cheap to enter (often $0.50-$1.50 for a $5-wide butterfly)
- Risk/reward can be 1:3 to 1:10
- Example: stock at $100, you think it stays near $100. Buy $95/$100/$105 call butterfly for $1.00. Max profit = $4.00 (if stock is exactly at $100 at expiration). Max loss = $1.00.
- Use for: pinning bets (stock stays near a price at expiration), cheap directional bets (skewed butterflies placed above current price for bullish bets), earnings plays with a specific target
- 7-21 DTE works best (you want the position to be near expiration for maximum payoff)

**Back-Ratio Spreads — Asymmetric Payoff**

- Sell 1 ATM call, buy 2 OTM calls (1:2 ratio) for a small debit or credit
- Profit if stock makes a large move in the direction of the long options
- Small loss if stock stays flat or moves slightly
- Example: stock at $100. Sell 1x $100 call for $5.00, buy 2x $105 calls for $2.80 each ($5.60 total). Net debit $0.60. If stock goes to $115: short call loses $15, two long calls gain $10 each = net $4.40 profit. If stock stays at $100: lose $0.60.
- Asymmetry: risk $0.60 to potentially make $5-$10+. But you need a large move.
- Best before events where a big move is likely but direction is uncertain — use call ratio for bullish, put ratio for bearish.

### What to Study

- **YouTube**: Adam Khoo "Advanced Options Strategies", tastylive "Butterfly" and "LEAPS" series, projectfinance debit spread tutorials
- **Books**: "Option Volatility and Pricing" by Sheldon Natenberg (chapters on spreads and volatility)
- **Tools**: Use ThinkorSwim Analyze tab to model P&L diagrams for every spread before entering

### What to Practice

Execute 10 directional options trades with specific thesis:
- 3 debit spreads (bull call or bear put) tied to technical breakouts or catalysts
- 2 LEAPS positions on stocks with 12-month conviction
- 2 poor man's covered calls (LEAPS + short call)
- 2 butterfly spreads (one ATM for pinning bet, one skewed for directional)
- 1 back-ratio spread before an event

For EACH trade, write your thesis BEFORE entering: why this stock, why this direction, why this strategy, what is the edge.

### How to Know You're Ready for Level 4

- You choose between debit and credit strategies based on IV rank automatically
- You can structure a PMCC and manage the short call through rolling
- You understand butterfly payoff diagrams intuitively
- You have 10 directional trades logged with thesis, result, and lessons
- You know when to use a debit spread vs a naked long option vs a LEAPS

### Common Mistakes at This Level

1. **Using debit spreads when IV is high** — you are overpaying. If IV rank > 50, favor credit strategies.
2. **LEAPS too far OTM** — a 0.50 delta LEAPS is a leveraged coin flip. Go deep ITM (0.80+ delta) for stock replacement.
3. **Not rolling PMCC short calls early enough** — if the short call hits 0.50 delta, roll it up and out. Don't wait until it's ITM.
4. **Butterflies too wide** — a $20-wide butterfly on a $100 stock will almost never hit max profit. Keep wings $5-$10 wide.
5. **Forgetting time** — debit spreads need TIME to work. 30 DTE is not enough for a thesis that needs earnings/catalysts. Use 45-60 DTE minimum.

---

## Level 4: Volatility Trading (Week 7-8)

### What You Need to Know

**Trading IV Itself, Not Direction**

This is where most retail traders never get to. Instead of predicting where a stock goes, you predict what happens to implied volatility. IV is mean-reverting — when it is extremely high, it tends to come down. When extremely low, it tends to go up. This is a statistical edge.

**Straddles and Strangles**

*Long straddle:* buy ATM call + ATM put (same strike, same expiration)
- Profit if stock moves significantly in either direction
- Cost: expensive (you're buying two ATM options)
- Break-even: strike +/- total premium paid
- When to buy: IV percentile < 25 (cheap options), you expect a big move (catalyst, breakout)
- When to sell: IV percentile > 60, you expect the stock to stay flat or move less than implied

*Short strangle:* sell OTM put + OTM call
- Profit if stock stays within a range
- Max profit: total premium received
- Risk: unlimited in theory (stock can move infinitely)
- When: IV rank > 50, no catalysts, range-bound stock
- Use defined-risk version (iron condor) until you have significant experience

**IV Crush Around Earnings**

This is one of the most reliable edges in options:
1. IV rises steadily into earnings as uncertainty builds
2. After the earnings announcement, uncertainty resolves
3. IV drops dramatically — often 30-50% overnight
4. This drop is called "IV crush"

How to profit:
- **Sell premium before earnings** (iron condor, short strangle, short straddle) — you profit from the IV drop even if the stock moves, as long as it stays within your range
- **Do NOT buy options before earnings** unless you are specifically playing the IV run-up (buy early, sell before announcement)
- The expected move (calculated from ATM straddle price) is exceeded only ~30% of the time — the market slightly overprices earnings moves on average

**VIX Options and Futures**

The VIX (CBOE Volatility Index) measures expected 30-day volatility of the S&P 500.

- VIX options are European-style, cash-settled, and settle on VIX futures (NOT spot VIX)
- VIX below 15 = complacent market, buy protective puts cheap
- VIX 15-20 = normal
- VIX 20-30 = elevated fear
- VIX 30+ = panic, historically great time to sell VIX calls or buy VIX puts (mean reversion)
- UVXY/SVXY for short-term VIX trades (but understand the decay in leveraged ETPs)

**Skew Trading**

Volatility skew = the difference in IV between OTM puts and OTM calls at the same distance from ATM.
- Normal skew: OTM puts have higher IV than OTM calls (markets crash faster than they rise, so put protection costs more)
- Steep skew = fear is elevated. The difference between OTM put IV and ATM IV is large.
- Flat skew = complacency. Puts are relatively cheap.
- Trading skew: when skew is steep, sell OTM puts (expensive) and buy ATM options (cheaper). When skew is flat, buy OTM puts for portfolio protection (cheap insurance).

**Vol Surface Analysis**

The volatility surface is a 3D map: strike price (x-axis) x expiration (y-axis) x implied volatility (z-axis).
- **Term structure**: how IV varies across expirations. Normally upward-sloping (longer-dated options have higher IV). When inverted (near-term IV > long-term IV), it signals an imminent event/fear.
- **Skew curve**: how IV varies across strikes for a single expiration
- When term structure is inverted: consider calendar spreads (sell expensive near-term, buy cheap long-term)
- When skew is extreme: consider risk reversals or skew trades

### What to Study

- **YouTube**: tastylive "Market Measures" series (data-driven vol analysis), Kai Zeng from tastylive on skew/vol surface, OptionAlpha volatility courses
- **Books**: "Option Volatility and Pricing" by Natenberg (THE bible for vol trading), "Volatility Trading" by Euan Sinclair (advanced, worth the effort)
- **Tools**: CBOE volatility charts (free), ThinkorSwim vol surface visualization, VIX Central for VIX futures term structure

### What to Practice

Trade 5 pure volatility plays:
1. One IV crush play around earnings (iron condor or short strangle on a mega-cap)
2. One long straddle/strangle when IV percentile < 20 on a stock nearing a catalyst
3. One calendar spread exploiting term structure inversion
4. One VIX-based trade (VIX call spread when VIX is at 12-13, or VIX put spread when VIX is at 30+)
5. One skew trade (sell expensive put skew via put spread, buy cheap calls)

### How to Know You're Ready for Level 5

- You check IV rank/percentile before EVERY trade and it determines your strategy choice
- You can look at a vol surface and identify term structure shape and skew
- You have traded IV crush around at least 3 earnings events
- You understand that selling options is selling insurance — you need a statistical edge (high IV) to profit long-term
- You stopped thinking purely in directional terms and started thinking in volatility terms

### Common Mistakes at This Level

1. **Selling vol when it's low** — IV rank < 20 and selling iron condors. The premium doesn't compensate you for the risk. You need IV rank > 40 minimum to sell.
2. **Fighting the VIX** — buying VIX puts when VIX is already at 14 (it can go lower). Selling VIX calls when VIX is at 22 (it can go much higher). Wait for extremes.
3. **Ignoring term structure** — selling a calendar spread when term structure is normal (near-term IV < long-term IV) means you are fighting the natural decay advantage.
4. **Overleveraging vol trades** — a short strangle can lose 5-10x the premium received. Size small.
5. **Not understanding VIX settlement** — VIX options settle to VIX Special Opening Quotation (SOQ), not spot VIX. They can diverge significantly. Trade VIX futures price, not spot.

---

## Level 5: Earnings Mastery (Week 9-10)

### What You Need to Know

**Expected Move Calculation**

The market prices in an expected move for earnings. Calculate it:
- Take the ATM straddle price (closest weekly expiration after earnings)
- Expected move = straddle price x 0.85 (adjustment for lognormal distribution)
- Example: AAPL at $200, ATM straddle costs $12. Expected move = $12 x 0.85 = ~$10.20, or about 5.1%.
- Stocks exceed the expected move only ~30% of the time. This is the house edge for premium sellers.

**Pre-Earnings Strategies**

1. **Buy IV run-up**: Buy options 2-3 weeks before earnings when IV is still relatively low. Sell them 1-2 days before earnings when IV has peaked. You don't care about the actual earnings — you're trading the IV increase.
   - Best on: stocks with consistent IV run-up patterns (most mega-caps)
   - Use ATM straddles or slightly OTM calls/puts
   - Exit BEFORE the announcement

2. **Sell IV at peak**: Sell iron condors or credit spreads the day before earnings when IV is highest.
   - Set strikes outside the expected move
   - Close the morning after earnings (take the IV crush profit)
   - Don't hold — the stock can keep moving post-earnings

**Post-Earnings Plays**

1. **Fade the gap**: if a stock gaps up/down more than the expected move on earnings, the move is often overdone. Sell premium in the direction of the gap (sell call spreads on a gap up, put spreads on a gap down) to fade the extreme.

2. **Continuation play**: if a stock gaps on earnings and the move is LESS than the expected move, it often continues in that direction for 2-5 days. Buy a debit spread in the direction of the gap with 2-3 week expiration.

3. **IV crush capture**: sell iron condors before earnings, buy them back the morning after at a steep discount due to IV crush. Even if the stock moves within the expected range, IV dropping 40% makes your position profitable immediately.

**Earnings Straddle Math**

When is a long straddle before earnings profitable?
- Only when the actual move exceeds the expected move significantly
- Historically, this happens ~30% of the time for mega-caps
- For smaller stocks with less analyst coverage, mispricings are more common (40-45%)
- Look for: stocks where recent realized volatility exceeds implied, upcoming earnings with new product launches or regulatory events not fully priced, stocks where the last 4 earnings moves all exceeded expected

**Sector Earnings Waves**

Earnings season has a predictable wave pattern:
1. Banks report first (JPM, BAC, GS) — sets risk sentiment
2. Then mega-cap tech (MSFT, GOOGL, META, AAPL, AMZN)
3. The first big tech report (usually MSFT or GOOG) sets the tone for the rest
4. If MSFT beats and gaps up, GOOGL/META/AAPL options get more expensive (IV rises) as the market expects similar beats
5. Trade this: buy calls/straddles on later-reporting stocks AFTER a strong first report in the sector

### What to Study

- **YouTube**: tastylive earnings studies (they have backtested data for every strategy), Kai Zeng earnings probability analysis, marketchameleon.com earnings data
- **Books**: No single book covers this well. Use marketchameleon.com for historical earnings data — expected move vs actual move for every stock, every quarter.
- **Tools**: marketchameleon.com (earnings expected move history), earningswhispers.com (earnings calendar), ThinkorSwim earnings tab

### What to Practice

Trade every mega-cap earnings in the current cycle:
- MSFT, GOOGL, META, AAPL, AMZN, NVDA, TSLA — that is 7 opportunities
- For each: calculate expected move, choose your strategy (sell or buy IV), document thesis
- Mix strategies: sell iron condors on 3, buy straddles on 2, play IV run-up on 2
- Track: actual move vs expected, IV before vs after, P&L, what you learned

### How to Know You're Ready for Level 6

- You can calculate expected move from an options chain in under 30 seconds
- You have traded both sides (buying and selling IV) around earnings and understand when each works
- You recognize sector earnings waves and use early reports to position for later ones
- You have a system for tracking earnings trades and their outcomes
- Your win rate on earnings trades exceeds 55%

### Common Mistakes at This Level

1. **Holding through earnings without a plan** — every earnings trade needs a pre-defined exit: close the morning after, or close at X profit/loss. No winging it.
2. **Selling too tight** — setting iron condor strikes inside the expected move to collect more premium. You will lose on the 70% of occasions where the stock moves within expected range but close to your strikes.
3. **Ignoring after-hours price action** — the stock moves after hours, but options don't trade until the next morning. A stock that gaps +8% after hours might open +5% or +11%. Your P&L is uncertain until the opening auction.
4. **Overconcentrating in one name** — don't put 10% of your portfolio on a single earnings trade. 1-3% max.
5. **Not accounting for pin risk** — if the stock lands near your short strike at expiration, you face assignment risk. Close positions before expiration if they're near the money.

---

## Level 6: Portfolio-Level Options (Week 11-12)

### What You Need to Know

**Portfolio Greeks Management**

Stop thinking about individual positions. Think about your PORTFOLIO Greeks:

- **Net delta**: sum of all position deltas (weighted by number of contracts and multiplier). A net delta of +500 means your portfolio moves like 500 shares of SPY. If you want to be neutral, net delta should be near zero.
- **Net theta**: how much your entire portfolio earns or loses per day from time decay. If you are a net premium seller, this should be positive (you earn theta daily).
- **Net vega**: your portfolio's exposure to changes in volatility. Positive vega = you profit when IV rises. Negative vega = you profit when IV falls. Most premium sellers are net negative vega.
- **Beta-weighted deltas**: convert all your deltas to SPY-equivalent. AAPL position with delta +200 and AAPL beta of 1.2 = +240 SPY-equivalent deltas. This lets you compare apples to apples across your portfolio.

Target portfolio Greeks:
- Net delta: slight positive bias (long-term bullish) or neutral
- Net theta: positive (you earn every day)
- Net vega: slightly negative (but not so negative that a vol spike wrecks you)
- Monitor and rebalance weekly

**Hedging with Index Options**

- Buy SPY or QQQ puts as portfolio insurance
- Collar your portfolio: if you are net long, buy OTM SPY puts and sell OTM SPY calls to fund them
- Cost of protection: rolling monthly 5% OTM SPY puts costs roughly 4-6% of portfolio value per year. That is expensive. Alternatives:
  - Buy 3-month puts and roll them (cheaper annualized)
  - Buy put spreads instead of outright puts (defined cost, partial protection)
  - Sell call spreads to fund put purchases (risk reversal)

**Collar Strategies**

For a position you own and want to protect without selling:
- Own 100 shares of AAPL at $200
- Buy $185 put (protection below $185) for $4.00
- Sell $215 call (give up upside above $215) for $4.00
- Net cost: $0 (the call premium funds the put)
- You keep the stock, you're protected below $185, you're capped at $215
- Zero cost collar = costless insurance with capped upside

**Risk Reversal for Leveraged Exposure**

- Sell an OTM put, use the premium to buy an OTM call (or call spread)
- Net cost: zero or near-zero
- Result: you have bullish exposure (long call) funded by downside risk (short put)
- Example: stock at $100. Sell $90 put for $3.00, buy $110 call for $3.00. Net cost $0. If stock goes to $120: call is worth $10, put expires worthless. If stock drops to $85: put is worth $5, call is worthless. P&L = -$5.
- Use for: stocks you are very bullish on and willing to buy on a dip. The short put acts as a commitment to buy at a discount.

**Dynamic Hedging**

As the market moves, your Greeks change. Dynamic hedging means adjusting:
- If your portfolio delta gets too positive (>800 SPY-equivalent), sell some delta: sell call spreads, buy put spreads, or reduce positions
- If your portfolio vega gets too negative (a vol spike would hurt), buy some vega: long straddles, VIX calls, calendar spreads
- Rebalance Greeks: weekly minimum, daily during volatile markets
- Think of it like steering a ship — small frequent adjustments, not big corrections

**Position Sizing with Kelly Criterion**

Kelly criterion tells you the optimal position size given your win rate and payoff ratio:
- f = (bp - q) / b
- f = fraction of portfolio to risk
- b = win/loss ratio (average win / average loss)
- p = probability of winning
- q = 1 - p
- Example: win rate 65%, average win $200, average loss $300. b = 200/300 = 0.67. f = (0.67 x 0.65 - 0.35) / 0.67 = 0.13/0.67 = 0.19. Optimal = 19% of portfolio. But this is the MAXIMUM — in practice, use half-Kelly (10%) or quarter-Kelly (5%) to account for estimation errors.
- For options specifically: your win rate and payoff ratio vary by strategy. Credit spreads might be 70% win rate with 1:2 payoff (small wins, big losses). Adjust accordingly.

### What to Study

- **YouTube**: tastylive portfolio management series, Predicting Alpha on portfolio Greeks, OptionAlpha portfolio builder
- **Books**: "Trading Options Greeks" by Passarelli (advanced Greeks management), "Dynamic Hedging" by Nassim Taleb (dense but transformative)
- **Tools**: ThinkorSwim portfolio analyze tab (shows aggregate Greeks), Interactive Brokers risk navigator

### What to Practice

Manage a 5-position options portfolio for 2 weeks:
- Position 1: a Wheel position (CSP or CC)
- Position 2: a credit spread (bull put or bear call)
- Position 3: a LEAPS or PMCC
- Position 4: an earnings play
- Position 5: a hedge (SPY/QQQ put spread or collar)

Each day:
- Calculate portfolio Greeks (net delta, theta, vega)
- Beta-weight to SPY
- Adjust if Greeks are out of balance
- Log daily Greek snapshots and adjustments made

### How to Know You're Ready for Level 7

- You think in portfolio Greeks, not individual position P&L
- You can beta-weight your entire book to SPY and know your market exposure in seconds
- You have managed a multi-leg portfolio through at least one volatile week
- You understand the cost of hedging and can evaluate which hedge method is most efficient
- You size positions using a systematic framework, not gut feel

### Common Mistakes at This Level

1. **Over-hedging** — hedging costs money. Every put you buy eats into returns. Hedge the tail risk, not every small dip.
2. **Ignoring correlation** — five positions in tech stocks don't give diversification. When NVDA drops, MSFT and GOOGL drop too. Beta-weight everything.
3. **Letting one position dominate** — if one position is 40% of your net delta, that is not a portfolio, that is one bet with four decorations.
4. **Not adjusting** — setting and forgetting is for shares, not options. Options positions change character daily. Check Greeks at least every other day.
5. **Full Kelly sizing** — Kelly assumes perfect knowledge of probabilities. You don't have perfect knowledge. Use half-Kelly or less.

---

## Level 7: Elite Execution (Ongoing)

### What You Need to Know

**Order Flow Reading**

Market makers provide liquidity. Understanding their behavior gives you edge:
- **Sweep orders**: large orders that sweep through multiple price levels. Bullish sweeps on calls = aggressive buying. Someone with conviction (or information) is paying up.
- **Dark pool prints**: large block trades on dark pools signal institutional activity. SPY dark pool prints at prices above the current market = bullish accumulation.
- **Options flow**: watch for unusual activity — single large trades (>$1M premium) on OTM options with short expiry. This is the closest to "smart money" signals.
- **Tools**: Unusual Whales, FlowAlgo, Cheddar Flow, or the free CBOE volume reports
- **Caveat**: 80% of unusual options flow is hedging, not speculation. A large put purchase could be a hedge on an existing long position. Context matters.

**Optimal Entry/Exit Timing**

- **Avoid the first 15 minutes**: bid-ask spreads are widest at open as market makers adjust. Options premiums are volatile and often mispriced (sometimes in your favor, usually not).
- **Best entry window**: 10:00-11:00 AM ET. Spreads have tightened, early morning volatility has settled, IV levels are more stable.
- **Best selling window for premium**: market open if IV spiked overnight (sell that inflated premium). Also end of day Friday (theta earns over the weekend).
- **Spread dynamics**: always use limit orders. Never market orders on options. Set your limit at the mid price, wait 30 seconds, adjust by $0.01-$0.02 if not filled. Patience saves $50-$200 per trade.
- **Closing auction**: if you need to close a position at end of day, submit your order by 3:50 PM ET. The last 10 minutes are chaotic.

**Rolling Positions**

Rolling = closing the current position and opening a new one at a different strike/expiration.

When to roll:
- **Winners at 50% profit**: close the credit spread, open a new one for the next cycle. Reset your theta clock.
- **Losers approaching max loss**: roll out in time (same strikes, later expiration) to collect more premium and give the trade more time.
- **Short calls being challenged**: roll up (higher strike) and out (later expiration) for a credit. If you can't roll for a credit, close the position.

When NOT to roll:
- Never roll a losing trade more than once. If the thesis is broken, take the loss.
- Never roll into an unfavorable IV environment (low IV rank on a credit trade).
- Never roll to avoid taking a loss your journal rules say you should take.

**Repair Strategies for Underwater Positions**

1. **Stock repair strategy**: you own shares at $100, stock is at $85. Buy 1x $85 call, sell 2x $92.50 calls (for free or a small credit). Break-even drops from $100 to $92.50. Tradeoff: gains are capped at $92.50.

2. **Spread repair**: your bull call spread is losing. Convert it to a butterfly by selling an additional spread above. Reduces max loss but also reduces max profit.

3. **Calendar repair**: a losing credit spread can sometimes be rolled into a calendar spread (close the short near-term, sell longer-term at same strike) to give the position more time.

4. **Accept the loss**: sometimes the best repair is no repair. A 1R loss is a cost of doing business. A 3R loss from trying to repair is a portfolio killer.

**Multi-Leg Adjustments Under Pressure**

When the market is moving fast against you:
- DO NOT panic-close multi-leg positions as individual legs. You will get terrible fills on each leg and the slippage will be more than the loss.
- Close the entire spread as a single order
- If you cannot get filled on the full spread, leg out starting with the short option (the one creating risk)
- Have contingent orders set up in advance: "if SPY drops below X, close this iron condor at market"
- Practice: use paper trading to simulate a fast market day and practice closing spreads quickly

**Building Your Own Vol Model**

The ultimate edge: your own IV forecast that differs from the market.
1. Calculate historical (realized) volatility for 10, 20, 30, 60 days
2. Compare to current IV — is IV overpriced (IV > realized vol) or underpriced?
3. Build a simple model: IV forecast = weighted average of recent realized vol + mean-reversion component + event premium
4. When your model says IV is significantly overpriced: sell premium. When underpriced: buy premium.
5. Track your model's accuracy vs the market's IV over time. If you're consistently better at forecasting vol than the market, you have a durable edge.

This is not easy. It takes months of data collection and analysis. But it is the same process professional vol traders at hedge funds use.

### What to Study

- **YouTube**: tastylive "From Theory to Practice" (Tom Sosnoff, the best practitioner), SMB Capital options desk videos, SqueezeMetrics for dark pool and GEX analysis
- **Books**: "Volatility Trading" by Euan Sinclair (vol modeling), "Trading and Exchanges" by Larry Harris (market microstructure), "Option Market Making" by Baird (how MMs think)
- **Tools**: ThinkorSwim thinkScript for custom vol analysis, ORATS for vol surface data, SqueezeMetrics GEX/DIX data, custom spreadsheets for tracking realized vs implied vol

### What to Practice

This is ongoing. Never stop:
- Build a spreadsheet tracking IV vs realized vol for 20 stocks over 60 days
- Record your entries: did you get filled at mid, worse than mid, better than mid? Track average slippage.
- Practice rolling positions in paper trading before doing it live
- Review order flow on 5 trades per week and assess whether the flow was predictive
- Build and backtest one vol model component per month

### How to Know You're At This Level

- You make money from execution edge alone (better fills than average)
- You have a systematic vol model, even if simple
- You can adjust a multi-leg position under pressure without panicking
- Your options P&L is consistently positive over 6+ month rolling periods
- You think about risk first, opportunity second
- You have a defined playbook and you stick to it

### Common Mistakes at This Level

1. **Overtrading** — at this level, you see setups everywhere. Discipline is taking only the best setups. If you're making more than 3-5 trades per day, you're probably forcing trades.
2. **Overconfidence** — a good streak leads to larger sizes which leads to one bad trade wiping out months of gains. Stay humble, stay sized right.
3. **Complexity for its own sake** — a 5-leg position that you can barely manage is worse than a simple 2-leg spread you understand cold. Complexity is not edge.
4. **Ignoring market regime** — your strategies from a low-vol trending market will not work in a high-vol choppy market. Adapt your playbook to the current regime.
5. **Not taking breaks** — options trading is mentally exhausting. One bad decision from fatigue can cost more than a week of good trading. Take days off.

---

## Summary: The Path

| Level | Focus | Key Skill | Duration |
|-------|-------|-----------|----------|
| 1 | Foundation | Understanding Greeks and IV | 2 weeks |
| 2 | Income | Selling premium profitably | 2 weeks |
| 3 | Directional | Structured bets with defined risk | 2 weeks |
| 4 | Volatility | Trading IV itself | 2 weeks |
| 5 | Earnings | Event-driven edge | 2 weeks |
| 6 | Portfolio | Managing aggregate risk | 2 weeks |
| 7 | Elite | Execution, flow, vol modeling | Ongoing |

Total time to competency: 12 weeks of focused study and paper trading. Total time to mastery: 2-5 years of live trading with continuous learning.

The edge is not in knowing the strategies. Everyone knows iron condors exist. The edge is in:
1. Knowing WHEN to use each strategy (IV rank dictates this)
2. Sizing correctly (surviving the inevitable losing streaks)
3. Managing positions actively (adjustments, rolls, Greeks balancing)
4. Execution (better fills, better timing, less slippage)
5. Emotional discipline (sticking to the plan when you want to deviate)

Do the work. Follow the levels. Paper trade until it's boring. Then go live small. Scale up only when you have evidence of consistent edge.
