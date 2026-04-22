---
type: market-research
agent: ORACLE
date: 2026-04-22
topic: options-trading-masterclass
tags: [options, trading, education, strategies, greeks, oracle, market]
---

# Options Trading Masterclass

> **ORACLE Deep Research** | Written for Presten — assumes solid stock knowledge, fresh eyes on options.
> Last updated: April 22, 2026

---

## Why Options Matter

Options aren't gambling instruments — they're precision tools. They let you define your risk in advance, generate income on positions you already hold, and profit from directions, timing, and volatility in ways stock-only investors can't. Every serious investor should understand them, even if they only use the simplest strategies.

The barrier to entry is jargon and complexity — this guide cuts through that.

---

## Part 1: Core Concepts

### What Is an Option?

An option is a **contract** that gives you the **right** (but not the obligation) to buy or sell 100 shares of a stock at a specific price by a specific date.

- **Call option** = the right to BUY 100 shares at the strike price
- **Put option** = the right to SELL 100 shares at the strike price

Every option contract has four components:

| Component | What It Is | Example |
|-----------|-----------|---------|
| **Underlying** | The stock the option is on | AAPL |
| **Strike price** | The price you can buy/sell at | $200 |
| **Expiration** | When the contract dies | May 16, 2026 |
| **Premium** | What you pay for the contract | $5.20 ($520 total for 100 shares) |

### Calls vs. Puts — The Mental Model

Think of options as insurance contracts:

- **Buying a call** = paying for the right to buy at a set price. You're bullish. You want the stock to go UP. Your max loss is the premium you paid.
- **Buying a put** = paying for the right to sell at a set price. You're bearish (or hedging). You want the stock to go DOWN. Your max loss is the premium you paid.
- **Selling a call** = collecting a premium in exchange for the obligation to sell shares if called upon. You're neutral-to-mildly-bullish. You want the stock to stay flat or go down slightly.
- **Selling a put** = collecting a premium in exchange for the obligation to buy shares if the price drops. You're neutral-to-bullish. You want the stock to stay flat or go up.

**Key insight:** Buying options gives you rights. Selling options gives you obligations. The seller collects a premium for taking on risk.

### Intrinsic vs. Extrinsic Value

Every option's price (premium) breaks into two pieces:

**Intrinsic value** = how much the option is "in the money" right now.
- A $200 call on a stock trading at $215 has $15 of intrinsic value.
- A $200 call on a stock at $195 has $0 intrinsic value (it's "out of the money").

**Extrinsic value** = everything else — time value + volatility premium.
- This is the "hope premium." It's what you pay for the chance that the option *could* become valuable.
- Extrinsic value decays to zero at expiration. This is called **time decay (theta).**

**Why this matters:** If you're buying options, time decay is your enemy — you're paying rent every day. If you're selling options, time decay is your friend — you collect rent every day.

### Moneyness

| Term | Call Example (Stock at $100) | Put Example (Stock at $100) |
|------|------------------------------|------------------------------|
| **In the money (ITM)** | Strike < $100 (e.g., $90 call) | Strike > $100 (e.g., $110 put) |
| **At the money (ATM)** | Strike = $100 | Strike = $100 |
| **Out of the money (OTM)** | Strike > $100 (e.g., $110 call) | Strike < $100 (e.g., $90 put) |

OTM options are cheaper but less likely to pay off. ITM options are expensive but behave more like the underlying stock. ATM options have the most extrinsic (time) value.

---

## Part 2: The Greeks

The Greeks measure how an option's price changes in response to different factors. You don't need to calculate them — your broker does that. You need to understand what they *mean* for your trade.

### Delta (Direction)

**What it measures:** How much the option price moves for each $1 move in the stock.

- Call delta: 0 to +1.0
- Put delta: 0 to -1.0
- ATM options: ~0.50 delta
- Deep ITM: ~0.90–1.0 delta (moves almost like the stock)
- Far OTM: ~0.05–0.15 delta (barely moves)

**Practical use:** Delta also roughly approximates the probability the option expires ITM. A 0.30 delta call has about a 30% chance of being in the money at expiration.

**Example:** You buy a 0.40 delta call for $3.00. Stock goes up $5. Your option goes up roughly $2.00 (0.40 x $5). Your call is now worth ~$5.00. That's a 67% return on a 5% stock move — this is the **leverage** of options.

### Gamma (Acceleration)

**What it measures:** How fast delta changes as the stock moves. It's the rate of change of delta.

- Highest for ATM options near expiration
- Low for deep ITM/OTM and long-dated options

**Practical use:** High gamma means your delta is unstable — your exposure is shifting rapidly. This is why selling short-dated ATM options is dangerous. Gamma is the reason "0DTE" (zero days to expiration) options can swing wildly.

**When you care about it:** If you sell options, gamma is your enemy. If the stock moves against you near expiration, gamma accelerates your losses. This is "gamma risk."

### Theta (Time Decay)

**What it measures:** How much value the option loses per day, all else equal.

- Theta is negative for option buyers (you lose value each day)
- Theta is positive for option sellers (you collect value each day)
- Theta accelerates as expiration approaches — the last 30 days are the fastest decay

**Practical use:** If you buy a call with theta of -$0.15, that option loses roughly $15 per day (per contract) in time value. Over a month, that's $450 gone just from time passing — the stock has to move enough to overcome that.

**Rule of thumb:** About 1/3 of an option's time value decays in the first 2/3 of its life. The remaining 2/3 of decay happens in the final 1/3 of the option's life. This is why selling 30-45 DTE (days to expiration) options is the sweet spot — maximum theta decay rate relative to risk.

### Vega (Volatility)

**What it measures:** How much the option price changes for each 1% change in implied volatility (IV).

- High vega = the option is very sensitive to volatility changes
- ATM options and long-dated options have the highest vega

**Practical use:** This is critical. You can be right about the direction and still lose money if volatility drops. Buying calls before earnings might seem smart, but if everyone else already bought calls, IV is inflated. After earnings, IV collapses ("IV crush"), and your option loses value even if the stock moves in your direction.

**The IV crush trap:** Stock up 3% after earnings, but your call loses 15% because IV dropped from 65% to 30%. This is the #1 way new options traders lose money.

**Implied Volatility Rank (IVR):** Measures current IV relative to its range over the past year. IVR > 50% means volatility is elevated — good time to sell options. IVR < 30% means volatility is cheap — better time to buy options.

### Rho (Interest Rates)

**What it measures:** How much the option price changes per 1% change in interest rates.

**Practical use:** With rates at ~4.35-4.50% as of April 2026, rho matters more than it did in the zero-rate era. Higher rates slightly increase call values and decrease put values. But for most retail trades, rho is the least important Greek — you can mostly ignore it unless you're trading LEAPS (long-dated options) where the interest rate component is meaningful.

### The Greeks Summary Table

| Greek | Measures | Buyer's Friend? | Seller's Friend? | Most Important When |
|-------|----------|-----------------|-------------------|---------------------|
| Delta | Direction | Yes (if right) | N/A | Always |
| Gamma | Delta acceleration | Yes (magnifies gains) | No (magnifies losses) | Near expiration |
| Theta | Time decay | No (costs money) | Yes (earns money) | Always, but especially < 30 DTE |
| Vega | Volatility sensitivity | Yes (if IV rises) | No (if IV rises) | Around earnings, events |
| Rho | Interest rate sensitivity | Minor | Minor | LEAPS only |

---

## Part 3: Strategies — From Simplest to Most Advanced

### Strategy 1: Covered Calls (Income Generation)

**What it is:** You own 100 shares of a stock. You sell a call option against those shares, collecting premium.

**When to use:** You're mildly bullish or neutral on a stock you already own. You want to generate income while waiting.

**Example:**
- You own 100 shares of AAPL at $210
- You sell a $225 call expiring in 30 days for $3.50 ($350 premium collected)
- **If AAPL stays below $225:** You keep the shares AND the $350. Repeat next month.
- **If AAPL goes above $225:** Your shares get called away (sold) at $225. You keep the $350 premium plus the $15/share gain. Total profit: $1,850. Not bad, but you miss any upside above $225.
- **If AAPL drops to $195:** You keep the $350 premium, which offsets $3.50/share of the loss. You still own the shares at a loss, but it's $350 less painful.

**The trade-off:** You cap your upside in exchange for immediate income. Covered calls work best on stocks you'd hold long-term anyway, where you're okay selling at the strike price.

**Ideal setup:** Sell 30-45 DTE, strike price at the level where you'd be happy selling. Collect 1-3% of the position value per month.

**Risk:** The stock crashes. The premium you collected doesn't offset a major drop. This is the same risk as holding the stock — the covered call just makes it slightly better.

---

### Strategy 2: Cash-Secured Puts (Getting Paid to Buy Low)

**What it is:** You sell a put at a strike price where you'd happily buy the stock. You keep enough cash in your account to buy 100 shares if assigned.

**When to use:** You want to own a stock but think it's slightly overpriced. You get paid to wait for your entry price.

**Example:**
- NVDA is at $155 and you think it's worth buying at $140
- You sell a $140 put expiring in 30 days for $4.00 ($400 premium)
- **If NVDA stays above $140:** You keep the $400 and never had to buy. Repeat.
- **If NVDA drops to $135:** You're assigned — you buy 100 shares at $140. But you collected $4/share, so your effective cost basis is $136. You wanted the stock at $140 anyway, and you got it cheaper.

**The math that matters:** Keep $14,000 in cash (100 shares x $140). Collect $400 premium. That's a 2.9% return on the cash in 30 days, whether or not you end up buying the stock. Annualized, that's a ~35% return if you keep repeating.

**Risk:** The stock crashes way below your strike. You're obligated to buy at $140 even if the stock is at $100. This is the same risk as placing a limit buy order at $140 — except you get paid while waiting.

---

### Strategy 3: Bull Call Spread (Cheaper Directional Bet)

**What it is:** Buy a call at a lower strike, sell a call at a higher strike, same expiration. You pay a net debit.

**When to use:** You're bullish but don't want to pay full price for a call. You're willing to cap your upside.

**Example:**
- SOL is at $215. You think it's going to $250 in the next 45 days.
- Buy the $220 call for $12.00
- Sell the $250 call for $4.00
- Net cost: $8.00 ($800 per contract)
- **Max profit:** $250 - $220 - $8 = $22 per share ($2,200). That's 175% return.
- **Max loss:** $8 per share ($800). This is your entire risk — defined and limited.
- **Break-even:** $228

**Why it's better than just buying a call:** The sold call offsets a chunk of the cost, reducing your break-even and the amount you can lose. Trade-off: you can't profit above $250.

### Strategy 4: Bear Put Spread (Cheaper Downside Bet)

**Mirror image of a bull call spread.** Buy a put at a higher strike, sell a put at a lower strike. You're bearish.

**Example:**
- Stock at $100. You think it drops to $85.
- Buy the $100 put for $6.00, sell the $85 put for $2.00
- Net cost: $4.00 ($400)
- Max profit: $100 - $85 - $4 = $11 per share ($1,100)
- Max loss: $400

---

### Strategy 5: Iron Condor (Bet on Sideways)

**What it is:** Sell a bull put spread AND a bear call spread simultaneously. You're betting the stock stays in a range.

**When to use:** You think the stock will trade sideways. IV is elevated (high IVR). You want to collect premium.

**Example:**
- Stock at $100. You think it stays between $90 and $110 for the next 30 days.
- Sell the $110 call, buy the $115 call (bear call spread)
- Sell the $90 put, buy the $85 put (bull put spread)
- Total credit collected: ~$2.50 ($250 per iron condor)
- **Max profit:** $250 if stock stays between $90 and $110
- **Max loss:** $500 - $250 = $250 (width of one spread minus premium)
- **Probability of profit:** Typically 60-75% depending on strike selection

**The sweet spot:** Iron condors work best when IVR is above 50%, because you're selling elevated premium that's likely to deflate. In low IV environments, the premium collected doesn't justify the risk.

**Current market note (April 2026):** With the VIX in a moderate range and earnings season creating sector-specific volatility spikes, iron condors can work well on individual stocks *after* their earnings reports (when IV has already crushed). Avoid running them into earnings — the gap risk destroys the strategy.

---

### Strategy 6: Straddles and Strangles (Bet on Big Movement)

**Straddle:** Buy both a call AND a put at the same strike price. You're betting the stock makes a big move in *either* direction.

**Strangle:** Buy an OTM call AND an OTM put (different strikes). Cheaper version of a straddle — needs a bigger move to profit.

**When to use:** You expect a big move but don't know the direction. Catalyst events like earnings, FDA decisions, legal rulings.

**Example (Straddle):**
- Stock at $100. Earnings in 5 days.
- Buy the $100 call for $4.50 and the $100 put for $4.00
- Total cost: $8.50 ($850)
- Break-even: stock needs to move above $108.50 or below $91.50
- That's an 8.5% move required just to break even

**The hard truth:** Straddles around known events are usually expensive because IV is already jacked up. The stock can move 5% and you still lose money because of IV crush. Straddles work best when you anticipate a big move that the *market* isn't expecting — which is rare.

**Better use case:** Long strangles when IV is unusually cheap (IVR < 20%) and you think volatility will increase. This is a volatility bet more than a direction bet.

---

### Strategy 7: The Wheel Strategy (The Income Machine)

**What it is:** A systematic combination of cash-secured puts and covered calls. It's a repeating cycle:

1. **Sell cash-secured puts** at a strike you'd buy the stock at. Collect premium.
2. **If assigned**, you now own the stock at a discount.
3. **Sell covered calls** on the shares you just acquired. Collect premium.
4. **If shares get called away**, you sold at a profit. Start over at step 1.

**Why it works:** You're getting paid at every step. You collect premium selling puts while waiting. You collect premium selling calls while holding. You buy stocks at a discount and sell at prices you're happy with.

**Ideal stocks for the Wheel:**
- Stocks you'd want to own long-term anyway
- Moderate to high IV (more premium to collect)
- Not prone to massive gap-downs (avoid pre-earnings, avoid biotech)
- Price range of $20-$150 (keeps capital requirements reasonable)

**Realistic returns:** 15-30% annualized on the capital deployed, depending on IV and stock selection. This won't make you rich overnight, but it's a serious income strategy.

**Risk:** The stock craters and you're stuck holding shares at a loss. The covered call premiums help, but they won't save you from a 40% drawdown.

---

### Strategy 8: LEAPS (Long-Term Calls as Stock Replacement)

**What it is:** Buying deep ITM call options with 1-2 years until expiration. LEAPS stands for Long-term Equity Anticipation Securities.

**When to use:** You're very bullish on a stock long-term but want leverage without the risk of short-dated options.

**Example:**
- MSFT is at $435. You want exposure but $43,500 for 100 shares is a lot of capital.
- Buy a $350 call expiring January 2028 (LEAPS) for ~$105 ($10,500)
- Delta: ~0.80. This call moves about 80% as much as the stock.
- You control 100 shares worth of MSFT for $10,500 instead of $43,500 — that's 4x leverage.
- **Time decay:** Because LEAPS are so long-dated, theta is minimal — maybe $5-10/day. You have years for the thesis to play out.

**Why this matters:** LEAPS give you stock-like exposure for a fraction of the capital. The freed-up capital can be deployed elsewhere. If MSFT goes to $535 in 18 months, your $10,500 LEAPS position is worth ~$18,500 — a 76% return vs. 23% for the stock.

**Risk:** If MSFT drops significantly and stays down, you lose your entire premium at expiration. Unlike stock, there's a time limit.

**Pro tip:** Buy LEAPS with at least 0.70 delta. Go deeper ITM for less leverage but more safety. Avoid OTM LEAPS — the time decay and probability of profit are against you.

---

### Strategy 9: Poor Man's Covered Call (PMCC)

**What it is:** A covered call strategy using a LEAPS call instead of owning 100 shares. You buy a long-dated deep ITM call (LEAPS) and sell short-dated OTM calls against it.

**When to use:** You want covered call income but don't have enough capital to buy 100 shares of expensive stocks.

**Example:**
- NVDA at $155. 100 shares = $15,500.
- Instead: Buy a Jan 2028 $120 LEAPS call for ~$48 ($4,800). Delta ~0.80.
- Sell a monthly $165 call for $3.50 ($350)
- **Capital deployed:** $4,800 vs. $15,500 for a traditional covered call
- **Monthly income:** $350 (7.3% of capital per month if repeated)
- **If NVDA goes above $165:** You can roll the short call up and out, or close the spread for a profit
- **If NVDA drops:** Your LEAPS loses value but you keep the $350. The LEAPS has years to recover.

**The key rule:** Your LEAPS must be deeper ITM than the short call's strike price, and your LEAPS expiration must be much longer than the short call's expiration. Never sell a short call that expires after your LEAPS.

**Risk:** A gap-down in NVDA eats into LEAPS value. Unlike owning shares, the LEAPS will eventually expire. You need to manage the position — roll the LEAPS before it has < 6 months to expiration.

---

## Part 4: Risk Management

### Position Sizing — The Most Important Skill

**Rule 1: Never risk more than 2-5% of your total portfolio on a single options trade.**

If your account is $25,000, your max risk per trade is $500-$1,250. This means:
- If you're buying options, spend no more than $500-$1,250 per trade
- If you're selling spreads, your max loss per spread should be within this range

**Rule 2: Diversify across time.** Don't put all your options trades on the same expiration date. Spread them across different expirations and underlyings.

**Rule 3: Know your max loss before you enter.** For every trade, calculate the worst-case scenario. If you can't stomach it, reduce position size.

### Max Loss Calculations by Strategy

| Strategy | Max Loss | How to Calculate |
|----------|----------|-----------------|
| Long call/put | Premium paid | Simple — you paid $500, you can lose $500 |
| Covered call | Stock drops to $0 minus premium | Same as owning stock, slightly buffered |
| Cash-secured put | Strike price x 100 minus premium | Like buying stock at the strike |
| Bull call / Bear put spread | Net debit paid | Spread cost = max loss |
| Iron condor | Width of one spread minus premium collected | ($5 wide spread - $2.50 collected) x 100 = $250 |
| LEAPS | Premium paid | Can be large — size accordingly |
| PMCC | LEAPS premium minus short call premiums collected | Manage before LEAPS expires |

### Rolling Positions

**Rolling** means closing your current option and opening a new one, usually at a later expiration and/or different strike. You roll to:

1. **Avoid assignment** when you don't want to be assigned on a short option
2. **Extend your time** when a long option needs more time to work
3. **Improve your position** by moving strikes up/down

**When to roll:**
- Short options: Roll when the option is ITM and you want to keep the position. Roll out in time (further expiration) for a credit if possible. Rolling for a debit means you're spending money to delay the inevitable — be honest about whether the trade thesis still holds.
- Long options: Roll when you have < 30-60 DTE and still believe in the thesis. Move to a later expiration to buy more time. This costs money.

**When NOT to roll:** If the original thesis is broken, close the position and take the loss. Rolling a losing trade repeatedly is how small losses become large losses.

### The 50% Rule

For premium-selling strategies (covered calls, cash-secured puts, iron condors): close the trade when you've captured 50% of the premium collected. Don't wait for expiration to squeeze out the last 50% — the risk/reward deteriorates dramatically in the final days, and a single bad move can wipe out weeks of profit.

**Example:** You collected $2.00 credit on an iron condor. When you can close it for $1.00, take the profit. Move to the next trade.

---

## Part 5: Current Market Context — April 2026

### What's Working Now

**Covered calls and cash-secured puts on quality names.** With the market in a "cautiously bullish" posture (see [[2026-04-22-market-scan]]), volatility is moderate — not spiking, not dead. This is the sweet spot for premium-selling strategies on stocks you'd want to own anyway. Targets:
- NVDA (high IV, earnings approaching — sell puts *after* earnings if the stock dips)
- MSFT (moderate IV, steady grinder — covered calls for income)
- AAPL (moderate IV, sideways-ish — perfect covered call candidate)

**Bull call spreads on BTC-related names.** With BTC approaching $100K (see [[Queue/pending-2026-04-21-btc-opportunity]]), names like MSTR, IBIT, and COIN have elevated IV that makes outright call buying expensive. Bull call spreads give you directional exposure while reducing the IV premium you're paying.

**LEAPS on high-conviction long-term holds.** With rates at ~4.4%, the cost of carry is real but manageable. If you're very bullish on NVDA, MSFT, or GOOG over 18+ months, LEAPS let you free up capital for other opportunities. The current moderate-IV environment means LEAPS aren't unreasonably expensive.

### What's Dangerous Now

**Buying short-dated calls into earnings.** Q1 earnings season is here. IV is inflated on everything reporting soon. Even if you're right about the direction, IV crush will eat your profits. If you want earnings exposure, use spreads (bull call, bear put) to reduce vega exposure.

**Selling naked puts on meme stocks or crypto tokens.** The crypto rally could pull memecoins and meme-adjacent equities (GME, AMC, MSTR) into volatile territory. Naked put selling on these names exposes you to gap-down risk that can exceed any premium collected.

**Iron condors on stocks with binary catalysts.** Don't sell iron condors on anything with earnings, FDA decisions, or major announcements coming. The gap risk destroys the defined-risk profile.

### Implied Volatility Environment

The VIX has been trading in a moderate range (~15-20), which tells us overall market IV is not extreme in either direction. Sector-specific IV is another story:
- **Tech (earnings season):** Elevated IV on individual names. Sell premium *after* earnings, not before.
- **Crypto-adjacent equities:** Elevated IV due to the BTC $100K narrative. Opportunities to sell premium here, but size small.
- **Utilities / Defensives:** Low IV. Not ideal for premium selling; better for LEAPS or outright stock positions.

---

## Part 6: Platform Recommendations

### For Beginners Learning Options

**Tastytrade (formerly tastyworks):** Built by options traders, for options traders. The best visualization of risk, probability of profit, and P&L curves. The research content (tastylive) is the best free options education on the internet. Commission: $1/contract to open, $0 to close.

### For Active Options Traders

**Thinkorswim (Charles Schwab):** The most powerful retail options platform. The analyze tab lets you model any strategy before you enter. Paper trading is excellent. Commission: $0.65/contract.

### For Mobile / Simplicity

**Robinhood:** Not the best fills, not the best analysis tools, but the interface is clean and the learning curve is low. Good for starting with basic covered calls and cash-secured puts. Commission: $0/contract (you pay in wider spreads).

### For Serious Traders

**Interactive Brokers (IBKR):** Best fills, lowest margin rates, most complete platform. The interface is ugly and complex, but the execution quality is the best in retail. If you're trading spreads and condors regularly, the fill improvement alone pays for itself. Commission: $0.65/contract or less.

**ORACLE's recommendation:** Start with Tastytrade for education and practice. Move to Thinkorswim or IBKR when you're comfortable.

---

## Part 7: Common Mistakes and How to Avoid Them

### Mistake 1: Buying OTM Weekly Options
**The trap:** They're cheap! $50 for a call that could be worth $500!
**The reality:** 85%+ of OTM weekly options expire worthless. You're donating money to market makers.
**The fix:** If you're buying options, buy at least 45-60 DTE with at least 0.30 delta. Give yourself time and reasonable probability.

### Mistake 2: Not Understanding IV Crush
**The trap:** Buying calls before earnings because you "know" the company will beat.
**The reality:** The stock beats, moves up 3%, and your call loses 20% because IV dropped 30 points.
**The fix:** Use spreads for earnings trades. Or wait until after the event and trade on the post-earnings move.

### Mistake 3: Letting Winners Turn Into Losers
**The trap:** Your covered call is at 70% profit. You wait for the last 30%. Stock gaps down. Profit gone.
**The fix:** The 50% rule. Take profits at 50% of max profit. Move to the next trade. Compounding small wins beats chasing big ones.

### Mistake 4: Over-Leveraging
**The trap:** Options are cheap! I can control 1,000 shares for what 100 shares would cost!
**The reality:** If the trade goes against you, you lose faster than you can react. Multiple losing trades can destroy an account.
**The fix:** 2-5% risk per trade. Maximum. No exceptions. The best options traders are disciplined position sizers, not big-bet gamblers.

### Mistake 5: Ignoring Assignment Risk
**The trap:** Selling options and forgetting that assignment is a real thing that happens.
**The reality:** If you sell a put, you might have to buy 100 shares. If you sell a call against shares you don't own (naked), your risk is unlimited.
**The fix:** Never sell options you can't cover. Cash-secured puts require the cash. Covered calls require the shares. Spreads have defined risk. Stick to these.

### Mistake 6: Trading Options on Low-Volume Underlyings
**The trap:** You find an option on a small-cap stock with seemingly great premium.
**The reality:** The bid-ask spread is $0.50 wide. You're giving up $50/contract just on the entry. You'll never get a fair price.
**The fix:** Stick to high-volume options: SPY, QQQ, AAPL, NVDA, MSFT, AMZN, META, TSLA, GOOG, AMD. These have penny-wide spreads and massive liquidity.

### Mistake 7: No Exit Plan
**The trap:** You enter a trade with a thesis but no defined exit — no profit target, no stop loss, no time-based exit.
**The fix:** Before every trade, write down: (1) profit target, (2) max loss you'll accept, (3) date you'll close regardless. Then follow the plan.

---

## Part 8: Quick Reference — Strategy Selector

| Market View | Strategy | Risk Level | Capital Needed |
|-------------|----------|------------|----------------|
| Bullish, own shares | Covered call | Low | 100 shares |
| Bullish, want to buy cheaper | Cash-secured put | Low-Medium | Strike x 100 cash |
| Bullish, defined risk | Bull call spread | Medium | Spread cost only |
| Bearish, defined risk | Bear put spread | Medium | Spread cost only |
| Sideways, high IV | Iron condor | Medium | Margin or spread width |
| Big move expected, any direction | Straddle/strangle | High | Premium cost |
| Long-term bullish, want leverage | LEAPS | Medium | LEAPS premium |
| Income on expensive stock | Poor man's covered call | Medium | LEAPS premium |
| Systematic income | Wheel strategy | Low-Medium | Cash for 100 shares |

---

## ORACLE's Options Playbook for April 2026

Based on current market conditions:

1. **Start with the Wheel on NVDA.** Wait for post-earnings. If NVDA dips, sell cash-secured puts at a strike 10-15% below the post-earnings price. If assigned, sell covered calls. Target: 2-3% monthly premium.

2. **Covered calls on any existing stock positions.** If you hold 100+ shares of anything in the portfolio (MSFT, AAPL, etc.), you should be selling covered calls 30-45 DTE, 5-10% OTM. This is free money on positions you'd hold anyway.

3. **Bull call spread on IBIT or MSTR** if the BTC-to-$100K thesis plays out. Define your risk, ride the momentum.

4. **LEAPS on MSFT** if you want long-term AI exposure without committing $43K+ for 100 shares. January 2028, deep ITM, 0.75+ delta.

5. **Avoid:** Straddles into earnings, iron condors on anything with an upcoming catalyst, any naked short options, and anything with less than 30 DTE unless you're experienced.

---

> This guide will be updated as market conditions change. Written by [[10 - Agents/ORACLE/Config|ORACLE]] for [[000 - Presten Manthey|Presten]].
> 
> **Disclaimer:** This is educational content and directional analysis, not licensed financial advice. Options involve risk of loss. Paper trade every strategy before using real capital.
