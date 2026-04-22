# Risk Management Bible

## The One Truth

**Risk management is the ONLY thing that separates long-term winners from losers.** Every other edge -- strategy, analysis, timing -- is secondary. You can have a 30% win rate and be profitable with proper risk management. You can have a 70% win rate and go broke without it.

---

## Position Sizing Methods

### 1. Fixed Percentage Risk (The Standard)
Risk a fixed percentage of your account on every trade.

**Formula**: Position Size = (Account Balance x Risk%) / (Entry Price - Stop Loss Price)

**Example**: $50,000 account, 1% risk, stock at $100, stop at $95.
- Risk amount: $50,000 x 0.01 = $500
- Dollar distance to stop: $100 - $95 = $5
- Position size: $500 / $5 = 100 shares

**Recommended risk per trade:**
| Trader Type | Risk Per Trade |
|------------|---------------|
| Conservative | 0.5% |
| Standard | 1% |
| Aggressive | 2% |
| Maximum (never exceed) | 3% |

**Why 1% works**: At 1% risk, you need to lose 100 trades in a row to blow up. Even a terrible 10-trade losing streak only costs 10% of your account -- recoverable.

### 2. Kelly Criterion (Mathematically Optimal)

**Formula**: K% = W - [(1-W) / R]
- W = win rate (as decimal)
- R = reward/risk ratio

**Example**: 55% win rate, 2:1 reward/risk
- K% = 0.55 - [(1-0.55) / 2] = 0.55 - 0.225 = 0.325 (32.5%)

**NEVER use Full Kelly.** Full Kelly maximizes growth but can produce 50-70% drawdowns.

**Practical Kelly usage:**
- Half Kelly (K/2) = 16.25% -- still aggressive
- Quarter Kelly (K/4) = 8.1% -- more manageable
- Most professionals use Quarter Kelly or less

**Requirements**: You need 50+ trades of data to estimate W and R accurately. With fewer trades, your estimates are unreliable and Kelly will give dangerous sizes.

### 3. Fixed Fractional (Simple and Effective)
Risk a fixed dollar amount per trade regardless of account size.
- Good for beginners building consistency
- Doesn't scale with account growth (adjust periodically)

### 4. Volatility-Adjusted Sizing
Use ATR (Average True Range) to adjust position size based on the asset's volatility.

**Formula**: Position Size = (Account x Risk%) / (ATR x Multiplier)

A volatile stock gets a smaller position. A calm stock gets a larger position. This normalizes risk across different assets.

---

## Stop Loss Strategies

### Hard Stop (Non-Negotiable)
Every trade must have a stop loss before entry. Period.

**Stop placement methods:**
- **Structure-based**: Below the last swing low (longs) or above the last swing high (shorts)
- **ATR-based**: Entry - (2 x ATR) for longs. Adapts to volatility.
- **Percentage-based**: Maximum 5% from entry for stocks, 3% for ETFs, 10% for crypto

### Trailing Stop
Once in profit, move your stop to lock in gains.
- Move to breakeven after 1R profit
- Trail by 2 ATR, or below the last swing low
- Never move a stop FURTHER from your entry (tightening only)

### When NOT to Use a Stop
- **Never** not use a stop. No exceptions.
- "Mental stops" don't count. They don't work because you'll rationalize holding.
- Options positions can use time-based exits or percentage-of-premium stops.

---

## Portfolio Heat (Total Exposure Management)

### Maximum Portfolio Risk
- **Maximum open risk**: 5-6% of total portfolio at any time
- With 1% risk per trade, this means 5-6 open positions maximum
- If all positions are correlated (same sector), reduce further

### Correlation Risk
- Don't hold 5 tech longs simultaneously -- they'll all move together
- Diversify across: sectors, asset classes (stocks + crypto), strategy types (momentum + mean reversion), timeframes
- In a market crash, correlations go to 1. All stocks drop together. Account for this.

### Drawdown Management
| Drawdown | Action |
|----------|--------|
| 0-5% | Normal. Continue trading full size. |
| 5-10% | Reduce position sizes by 50%. Review recent trades for errors. |
| 10-15% | Reduce to paper trading. Full strategy review. |
| 15-20% | Stop trading. Complete strategy audit before returning. |
| 20%+ | Something is fundamentally broken. Don't trade until you identify and fix the problem. |

---

## Risk/Reward Math

### The Math of Expectancy
**Expectancy** = (Win Rate x Average Win) - (Loss Rate x Average Loss)

| Win Rate | Required R:R to Break Even | Required R:R to be Profitable |
|----------|---------------------------|------------------------------|
| 30% | 2.33:1 | 3:1+ |
| 40% | 1.5:1 | 2:1+ |
| 50% | 1:1 | 1.5:1+ |
| 60% | 0.67:1 | 1:1+ |
| 70% | 0.43:1 | 0.75:1+ |

**The key insight**: You don't need a high win rate. You need favorable math. A 40% win rate with 3:1 R:R is more profitable than a 60% win rate with 0.5:1 R:R.

### Minimum R:R Requirements
- **Never** take a trade with less than 1:1.5 R:R
- Ideal minimum: 1:2 R:R
- Swing trades should target 1:2 to 1:3
- Day trades can accept 1:1.5 to 1:2

---

## The Math of Ruin

### How Losses Compound Against You
| Portfolio Loss | Gain Needed to Recover |
|---------------|----------------------|
| 10% | 11% |
| 20% | 25% |
| 30% | 43% |
| 40% | 67% |
| 50% | 100% |
| 60% | 150% |
| 70% | 233% |

**A 50% loss requires a 100% gain to recover.** This is why protecting capital is MORE important than making gains. The math of recovery is asymmetric and works against you.

---

## Rules You Must Follow

1. **1% max risk per trade** (2% absolute maximum for highest-conviction setups)
2. **5% max portfolio heat** (total open risk at any time)
3. **Always use a stop loss** (placed before entry, never moved further away)
4. **Never add to a losing position** (averaging down is how accounts die)
5. **Cut losses fast, let winners run** (the opposite of what feels natural)
6. **No revenge trading** (loss limits prevent this: 2% daily max loss, stop trading)
7. **Diversify across correlations** (not just tickers, but actual correlation)
8. **Scale down on drawdowns** (50% size reduction at 5% drawdown)
9. **Keep a trading journal** (review every trade, identify patterns in your mistakes)
10. **Risk/reward must be 1:1.5 minimum** before entering any trade

---

## ORACLE's Verdict
- **Edge:** This IS the edge
- **Difficulty:** Easy to understand, Hard to execute consistently
- **Best for:** Every trader, every strategy, every market
- **Win rate (estimated):** N/A -- risk management is the foundation, not a strategy
- **Risk/Reward:** N/A -- it defines your risk/reward on every trade
- **Will I use this?** Non-negotiable. Every single trade.
- **Why:** Risk management is the single most important skill in trading. Full stop. Every successful trader I've studied says the same thing: the strategy barely matters compared to how you manage risk. The 1% rule, the 5% portfolio heat limit, the drawdown scaling -- these aren't suggestions, they're survival rules. The math of ruin shows why: a 50% loss requires 100% to recover. The asymmetry of losses means capital preservation must come before capital appreciation. This is the foundation everything else is built on.
