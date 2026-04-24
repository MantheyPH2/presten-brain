# ORACLE Portfolio Stress Test

> **Run every Sunday + before any major position increase**
> "If the market crashes right now, what happens to my portfolio?" If you can't answer this instantly, you're flying blind.

---

## What Is a Stress Test

A stress test simulates how your portfolio performs under extreme but plausible market conditions. It's a fire drill — you want to find the weaknesses BEFORE the fire, not during it.

**Why it matters:**
- The 2020 COVID crash took SPY down 34% in 23 trading days
- The 2022 bear market took QQQ down 37% over 9 months
- April 2025 "Liberation Day" tariffs crashed markets 10%+ in days
- The 2026 Iran war escalation has already caused 5-8% drawdowns
- Every portfolio that survived had a plan. Every one that blew up didn't.

---

## Stress Test Matrix

For every open position, calculate the impact at each crash level.

### How to Fill This Out

| Position | Ticker | Size ($) | Current P&L | Beta | If SPY -5% | If SPY -10% | If SPY -15% | If SPY -20% |
|----------|--------|---------|-----------|------|-----------|------------|------------|------------|
| Example: NVDA shares | NVDA | $5,000 | +$800 | 1.65 | -$412 | -$825 | -$1,237 | -$1,650 |
| Example: SPY puts (hedge) | SPY P | $500 | -$50 | -1.0 | +$250 | +$500 | +$750 | +$1,000 |
| Example: BTC | BTC | $3,000 | +$200 | ~1.8 | -$270 | -$540 | -$810 | -$1,080 |
| Position 1 | ____ | $____ | $____ | ____ | $____ | $____ | $____ | $____ |
| Position 2 | ____ | $____ | $____ | ____ | $____ | $____ | $____ | $____ |
| Position 3 | ____ | $____ | $____ | ____ | $____ | $____ | $____ | $____ |
| **TOTAL** | | **$____** | **$____** | | **$____** | **$____** | **$____** | **$____** |

---

## How to Calculate Each Asset Class

### Stocks
**Formula:** Position Loss = Position Size x Beta x Market Drop

| Beta | What It Means | If SPY -10% | Examples |
|------|--------------|------------|---------|
| 0.5 | Moves half as much as market | Stock drops ~5% | JNJ, PG, KO |
| 1.0 | Moves with market | Stock drops ~10% | SPY, most large caps |
| 1.5 | Moves 1.5x market | Stock drops ~15% | NVDA, TSLA, AMD |
| 2.0 | Moves 2x market | Stock drops ~20% | ARKK, small cap growth |
| 2.5+ | Moves 2.5x+ market | Stock drops ~25%+ | Meme stocks, micro caps |

**Where to find beta:** Yahoo Finance quote page, or `WebSearch "[ticker] beta"`

**Important nuance:** Beta is a historical average. In a crash, correlations spike and everything drops together. High-beta stocks often drop MORE than beta predicts in a panic.

### Options
Options are non-linear. You can't just multiply by beta. Use delta and gamma.

**Long Calls:**
- Loss = Delta x Underlying Move x 100 (per contract)
- Example: NVDA $150 call, delta 0.45, NVDA drops 15% ($22.50)
- Loss per contract = 0.45 x $22.50 x 100 = $1,012
- But: delta increases as stock drops (gamma), so actual loss is WORSE
- Rule of thumb: In a crash, assume you lose 60-80% of long call value

**Long Puts (hedges):**
- GAIN = Delta x Underlying Move x 100 (per contract)
- These INCREASE in value during crashes — this is your protection
- Rule of thumb: Assume puts gain 2-4x their value in a -10% crash (delta expansion + IV spike)

**Spreads:**
- Max loss is defined. Use max loss as worst case.
- Bull call spreads: assume max loss on any move beyond the short strike
- Bear put spreads: these PROFIT in a crash — calculate gain at max profit level

**Short Options (naked):**
- EXTREME RISK. Calculate as if underlying moves to your short strike.
- Naked calls in a melt-up or naked puts in a crash = unlimited loss potential
- Margin calls come FAST. Brokers liquidate at the worst prices.

### Crypto
Crypto has no beta in the traditional sense, but historical correlation to SPY during crashes is well-documented.

| Asset | Historical Drop vs SPY Drop | In a -10% SPY crash | In a -20% SPY crash |
|-------|---------------------------|---------------------|---------------------|
| BTC | 1.5-2.0x | -15% to -20% | -30% to -40% |
| ETH | 2.0-2.5x | -20% to -25% | -40% to -50% |
| SOL | 2.5-3.5x | -25% to -35% | -50% to -70% |
| Alts/Memes | 3.0-5.0x | -30% to -50% | -60% to -100% |

**Key insight:** In a true risk-off event, crypto crashes FIRST and HARDEST. The 24/7 nature means it's the first thing liquidated. Don't assume crypto is "uncorrelated" — during crashes, everything correlates to 1.

### Inverse Positions / Hedges
These are your parachute. Calculate how much they GAIN.

| Hedge Type | What It Does in a Crash | Sizing Guide |
|-----------|------------------------|-------------|
| SPY puts | Gain 2-4x value in -10% crash | 2-5% of portfolio in puts = meaningful hedge |
| VIX calls | VIX doubles in crashes — calls explode | 0.5-1% of portfolio — high leverage |
| SQQQ (3x inverse QQQ) | Gains 30% when QQQ drops 10% | Small position — decays when wrong |
| SH (inverse SPY) | Gains ~10% when SPY drops 10% | Moderate position — less decay |
| Cash | Doesn't lose value. Provides buying power. | 10-30% cash = ultimate hedge |
| Treasury bonds (TLT) | Usually rallies in equity crashes | 5-15% allocation — rate sensitive |
| Gold (GLD) | Safe haven — rallies in systemic crises | 5-10% allocation |

### Net Portfolio Impact
**Total Impact = Sum of all position losses + Sum of all hedge gains**

If your net impact at -10% is worse than -15% of your portfolio, you're TOO AGGRESSIVE. Either reduce positions or add hedges.

---

## 5 Stress Test Scenarios

### Scenario 1: Mild Pullback (-5%)
**Triggers:** Earnings disappointment from mega-cap, hot CPI print, hawkish Fed comment, minor geopolitical escalation
**Historical precedent:** Happens 3-4 times per year. Normal market function.
**What happens:**
- SPY drops 5% over 3-7 trading days
- VIX spikes to 22-28
- Growth stocks drop 7-10%
- Crypto drops 8-15%
- Bonds rally slightly
- Recovery typically within 2-4 weeks

**ORACLE Action Plan:**
- DO NOTHING if portfolio loss is under 7%. This is noise.
- Review stops on weakest positions
- If VIX spikes above 25, SELL premium (iron condors, put spreads) on quality stocks
- Begin building watchlist for "buy the dip" entries
- Do NOT add new long exposure until pullback stabilizes (2-3 days of no new lows)

### Scenario 2: Correction (-10%)
**Triggers:** Geopolitical escalation (Iran war expansion), credit event, recession data, major earnings misses across multiple sectors
**Historical precedent:** Happens 1-2 times per year. Painful but not catastrophic.
**What happens:**
- SPY drops 10% over 2-6 weeks
- VIX spikes to 28-35
- Growth stocks drop 15-20%
- Crypto drops 20-30%
- Flight to quality — bonds, gold, USD rally
- Recovery typically 1-3 months

**ORACLE Action Plan:**
- Cut any position down more than 15% with no catalyst for recovery
- Close all short-term speculative trades
- Increase hedges to cover 30-50% of downside
- Sell calls against long positions (convert to covered calls)
- Begin ACCUMULATING high-conviction long-term names at lower prices
- Increase cash to 20-30% of portfolio
- Check portfolio daily, but don't panic trade

### Scenario 3: Crash (-15% to -20%)
**Triggers:** Black swan — Liberation Day style tariffs, banking crisis, pandemic, military escalation beyond Iran, sovereign debt crisis
**Historical precedent:** Happens every 3-5 years (2018, 2020, 2022, 2025)
**What happens:**
- SPY drops 15-20% over 1-8 weeks
- VIX spikes to 35-50+
- Growth stocks drop 25-40%
- Crypto drops 40-60%
- Margin calls cascade. Forced selling creates overshoots.
- Recovery: 3-12 months

**ORACLE Action Plan:**
- IMMEDIATELY cut all leveraged and speculative positions
- Close ALL short options positions that can be covered (margin requirements spike)
- Deploy remaining hedges — exercise puts if deep ITM
- Raise cash to 40-50% of portfolio
- Do NOT try to catch the falling knife in the first 3 days
- After day 3-5 of crash, begin scaling into the highest-quality names (AAPL, MSFT, GOOGL, JPM)
- Buy in 3-5 tranches over 2-4 weeks — never all at once
- This is where fortunes are made. But only if you have cash and didn't blow up.

### Scenario 4: Flash Crash (-5% in 1 day)
**Triggers:** Japan carry trade unwind (Aug 2024 style), liquidity crisis, algorithmic cascading, options expiration + news catalyst
**Historical precedent:** 2010 Flash Crash, Aug 2024 carry trade unwind, 2015 China devaluation
**What happens:**
- SPY drops 5%+ intraday, often in 30-90 minutes
- VIX spikes 50-100% intraday
- Circuit breakers may trigger (Level 1 at -7%, Level 2 at -13%, Level 3 at -20%)
- Bid/ask spreads blow out. Limit orders skip. Stop losses trigger at terrible prices.
- Recovery often begins same day or next day

**ORACLE Action Plan:**
- DO NOT SELL INTO A FLASH CRASH. This is the worst time to sell.
- If stops trigger, let them. Don't chase.
- Wait for VIX to peak and start declining before acting
- If you have cash, flash crashes are the single best buying opportunity
- After the crash: review what happened, adjust position sizing for volatility

### Scenario 5: Crypto-Specific Crash (-30% to -50%)
**Triggers:** Exchange hack/insolvency (FTX 2.0), regulatory crackdown, stablecoin depeg (USDT/USDC losing peg), major protocol exploit
**Historical precedent:** FTX collapse (-65% BTC), LUNA/UST depeg (-80% market), China mining ban
**What happens:**
- BTC drops 30-50% over 1-4 weeks
- ETH drops 40-60%
- Alts drop 60-90%
- DeFi protocols face cascading liquidations
- Stablecoins may temporarily lose peg
- Traditional markets may be unaffected (crypto-isolated) or slightly impacted

**ORACLE Action Plan:**
- IMMEDIATELY move all crypto off centralized exchanges to cold storage
- Check stablecoin exposure — if USDT/USDC depegs, convert to fiat ASAP
- Close all leveraged crypto positions (perps, margin)
- If exchange-specific: withdraw from that exchange immediately
- Do NOT sell BTC below the 200-week moving average — historically this is the max pain bottom
- After dust settles (1-2 weeks): accumulate BTC only. Alts may not recover.
- Verify on-chain that exchange reserves haven't been drained (Glassnode)

---

## The Red Line System

Set hard rules for when to act. Don't rely on judgment during panic.

| Portfolio Loss | Status | Action |
|---------------|--------|--------|
| -3% | YELLOW | Review positions. Check stops. No panic. |
| -5% | ORANGE | Cut weakest 20% of positions. Add hedges. |
| -8% | RED | Cut all speculative positions. Move to 30% cash. |
| -12% | CRITICAL | Emergency mode. Cut to core only (top 3 positions). 50% cash. |
| -15%+ | SURVIVAL | All options, all specs closed. Only BTC, blue chips, and cash. |

---

## Weekly Stress Test Template

Run every Sunday evening. Takes 15 minutes. Saves your portfolio.

```
=== ORACLE PORTFOLIO STRESS TEST ===
Date: ____
Week #: ____

--- PORTFOLIO SNAPSHOT ---
Total portfolio value: $____
Cash position: $____ (___% of portfolio)
Number of open positions: ____
Largest single position: ____ ($____, ___% of portfolio)

--- STRESS SCENARIOS ---
If SPY -5%:
  Portfolio becomes: $____ (___% loss = $____)
  Largest position loss: $____ on ____
  Hedge offset: +$____
  Net impact: -$____

If SPY -10%:
  Portfolio becomes: $____ (___% loss = $____)
  Largest position loss: $____ on ____
  Hedge offset: +$____
  Net impact: -$____

If SPY -15%:
  Portfolio becomes: $____ (___% loss = $____)
  Largest position loss: $____ on ____
  Hedge offset: +$____
  Net impact: -$____

--- CONCENTRATION RISK ---
Top position as % of portfolio: ____%  (TARGET: <20%)
Top 3 positions as % of portfolio: ____%  (TARGET: <50%)
Single sector exposure: ____%  (TARGET: <30%)
Crypto as % of portfolio: ____%  (TARGET: <25%)
Leverage ratio: ____x  (TARGET: <1.5x)

--- HEDGE ANALYSIS ---
Current hedges in place: ____
Hedge cost (monthly): $____
Downside protection coverage: ____%
Is protection adequate? YES / NO
Cost of adding full protection: $____

--- CORRELATION CHECK ---
Are positions correlated? (If SPY drops, do ALL positions drop?)
Correlated positions: ____
Uncorrelated/inverse positions: ____
Diversification score (1-10): ____

--- ACTION ITEMS ---
Positions to reduce: ____
Hedges to add: ____
Cash target change: ____
Risk level: ACCEPTABLE / NEEDS ADJUSTMENT / DANGEROUS

--- MAX DRAWDOWN TOLERANCE ---
Maximum acceptable portfolio loss: $____ (___%)
Current max drawdown at -15%: $____ (___%)
Buffer remaining: $____
```

---

## Position Sizing from Stress Test

Use the stress test to SET position sizes, not just analyze them.

**Rule:** No single position should cause more than 3% portfolio loss in a -10% market crash.

```
Max Position Size = (Portfolio Value x 3%) / (Beta x 10%)

Example:
- Portfolio: $50,000
- Max single position loss at -10% crash: $1,500 (3%)
- For NVDA (beta 1.65): Max size = $1,500 / (1.65 x 0.10) = $9,090
- For BTC (effective beta 1.8): Max size = $1,500 / (1.8 x 0.10) = $8,333
- For SOL (effective beta 3.0): Max size = $1,500 / (3.0 x 0.10) = $5,000
```

---

## Related ORACLE Files
- [[risk-dashboard]]
- [[pre-trade-checklist]]
- [[options-setups-playbook]]
- [[sector-rotation-model]]
- [[pre-market-scanner]]

---

*Sources: [Morningstar Stress Testing](https://www.morningstar.com/business/insights/blog/portfolio-construction/portfolio-stress-testing), [BlackRock Scenario Tester](https://www.blackrock.com/us/financial-professionals/tools/analyze-portfolio-risk), [Phoenix Strategy Group Stress Test Scenarios](https://www.phoenixstrategy.group/blog/5-scenarios-to-stress-test-portfolio-volatility), [Waterloo Capital Stress Test Guide](https://waterloocap.com/portfolio-stress-testing-guide/), [WisdomTree Stress Test Methodology](https://www.wisdomtree.com/investments/tools/stress-test-methodology)*
