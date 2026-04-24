---
tags: [oracle, trading, options, greeks, delta, gamma, theta, vega]
created: 2026-04-21
---

# Greeks Calculator

> Options without understanding the Greeks are lottery tickets. Options with Greek mastery are a precision instrument for expressing exactly the view you want with exactly the risk you're willing to take.

---

## The Four Greeks That Matter

Delta, Gamma, Theta, and Vega drive 99%+ of all option price movement in normal conditions. Rho matters only in specific long-dated situations. Master these four and you understand your positions.

**Key principle:** The Greeks are not independent. They interact constantly. When price moves, Delta and Gamma change together. As time passes, Theta erodes premium while Gamma accelerates. When volatility shifts, Vega reshapes every position. You must think about them as a system, not in isolation.

---

## Delta — Directional Exposure

### What It Measures
How much the option's price changes for each $1 move in the underlying stock.

**Call options:** Delta is positive (0 to +1.0) — calls gain value when stock rises
**Put options:** Delta is negative (0 to -1.0) — puts gain value when stock falls

### Quick Reference by Moneyness

| Option Type | Delta Range | $1 Stock Move = |
|-------------|-------------|----------------|
| Deep ITM call | +0.80 to +0.95 | Option gains $0.80–$0.95 |
| ATM call | +0.45 to +0.55 | Option gains $0.45–$0.55 |
| Slightly OTM call | +0.30 to +0.45 | Option gains $0.30–$0.45 |
| Far OTM call | +0.05 to +0.20 | Option gains $0.05–$0.20 |
| Deep ITM put | -0.80 to -0.95 | Option gains $0.80–$0.95 on stock drop |
| ATM put | -0.45 to -0.55 | Option gains $0.45–$0.55 on stock drop |
| Far OTM put | -0.05 to -0.20 | Option gains $0.05–$0.20 on stock drop |

### Delta as Probability Proxy

Delta also approximates the **probability of expiring in-the-money:**
- Delta 0.70 call → roughly 70% probability of expiring ITM
- Delta 0.30 call → roughly 30% probability of expiring ITM
- Delta 0.50 (ATM) → coin flip

This is not mathematically perfect but is an excellent mental shorthand.

### Portfolio Delta

Net portfolio delta tells you your directional exposure to the underlying or to the market:

```
Net Delta Calculation:
  Long 2 calls with delta 0.50 = +1.00 delta
  Short 1 put with delta -0.40 = +0.40 delta (short put = positive delta)
  Long 100 shares = +1.00 delta per share = +100 delta
  
  Total: +101.40 delta
  
  Interpretation: For every $1 the underlying rises,
  this portfolio gains approximately $101.40
```

**Delta-neutral:** Net delta near zero. The position doesn't care about small directional moves — it profits from volatility, time, or IV changes instead.

### Mental Math: Delta in Sizing

When buying options for directional exposure:
- Want the leverage of $1,000 in stock? Buy calls with delta equivalent to ~$1,000 notional
- Example: Stock at $100 × delta 0.50 = $50 per $1 move × 1 contract (100 shares) = $50/move
- Equivalent to holding $5,000 worth of stock (50 shares at $100)

---

## Gamma — The Accelerator

### What It Measures
How much **Delta changes** for each $1 move in the underlying. Gamma is the rate of change of Delta.

If Delta is velocity, Gamma is acceleration.

### Quick Reference

| Scenario | Gamma Character |
|----------|----------------|
| ATM option | Highest gamma (delta changes fastest) |
| Deep ITM or Far OTM | Low gamma (delta already near 1.0 or 0.0) |
| Near expiration | Gamma spikes dramatically for ATM options |
| Far from expiration | Gamma is lower, more stable |

**Example:**
- You hold an ATM call with Delta 0.50 and Gamma 0.10
- Stock rises $1 → Delta becomes 0.60 (gained 0.10 from gamma)
- Stock rises another $1 → Delta becomes 0.70 (gained another 0.10)
- Your calls are now acting like they have 70% as much exposure as owning the stock

### Long Gamma vs. Short Gamma

**Long gamma (bought options):**
- You benefit from large moves in either direction
- The bigger the move, the more your delta accelerates in your favor
- You're paying theta (time decay) for this benefit

**Short gamma (sold options):**
- You lose money on large moves in either direction
- A big move against you causes your delta to work harder against you
- You're collecting theta as compensation for this risk
- **Dangerous near expiration:** Gamma spikes for ATM options in the final days. Short gamma into expiration = significant pin risk and rapid delta change

### Gamma Risk Near Expiration

In the final 7 days, ATM options experience dramatically elevated gamma. A stock that moves $2 in a day can swing a short ATM option's delta from 0.50 to 0.90 — almost like owning the stock. If you're short those options, your theoretical loss can double or triple compared to the same move two weeks prior.

**ORACLE rule:** Close or roll short options positions when they're ATM and within 7 DTE (days to expiration) unless there's a specific strategy reason to hold.

---

## Theta — The Passage of Time

### What It Measures
How much the option's price declines with each day that passes, all else equal. Theta is always negative for long options (you lose value every day) and positive for short options (you collect value every day).

### Quick Reference

| Scenario | Theta Character |
|----------|----------------|
| ATM options | Highest theta decay (most extrinsic value to lose) |
| Deep ITM or Far OTM | Lower theta (less extrinsic value) |
| Near expiration (< 30 DTE) | Theta accelerates — decay is fastest |
| Far expiration (90+ DTE) | Theta is smaller and more stable per day |

**Theta is non-linear.** An option that decays $0.02 per day at 60 DTE might decay $0.08 per day at 7 DTE.

### The Theta Decay Curve

```
Time Remaining → Rate of Decay (approximate, ATM option):
  90 DTE   → slow decay (~1/3 of final-week rate)
  60 DTE   → moderate decay
  30 DTE   → accelerating (3-5× faster than at 90 DTE)
  14 DTE   → fast decay
  7 DTE    → rapid decay (~3-5× faster than 30 DTE)
  1 DTE    → extreme — option loses most remaining extrinsic value
```

**Rule of thumb:** An ATM weekly option (5 DTE) loses approximately 1/5 of its remaining extrinsic value per day in the final week.

### Theta as Friend or Enemy

**Long options (bought calls or puts):**
- Theta is your **enemy** — you are bleeding premium every single day the underlying doesn't move enough
- Break-even rule: a long option must move enough to overcome its daily theta cost. An ATM option with $0.10/day theta needs the stock to move enough to gain $0.10+ in delta profits just to tread water
- Solution: Buy options with more time than you think you need (at least 2× your expected trade duration)

**Short options (sold premium — iron condors, covered calls, CSPs, short strangles):**
- Theta is your **friend** — you collect time decay every day the underlying stays within your range
- The theta income is compensation for the risk you're taking
- Ideal conditions: high IV (sell premium when it's richest), stable underlying

### Practical Theta Math

```
Daily theta check (long positions):
  If your long call has theta -$0.08:
  → You need the stock to move enough to gain $0.08+ in delta P&L just to break even today
  → Over a week: you need $0.56 in delta gains just to stay flat
  → The stock needs to move (theta per day ÷ delta) = $0.08 ÷ 0.50 = $0.16/day just to offset theta
```

---

## Vega — The Volatility Variable

### What It Measures
How much the option's price changes when **implied volatility (IV)** changes by 1 percentage point.

All options gain value when IV rises and lose value when IV falls. Vega is always positive for long options and negative for short options.

### Quick Reference

| Scenario | Vega Character |
|----------|---------------|
| Long options (any) | Positive vega — you benefit from IV expansion |
| Short options (any) | Negative vega — you benefit from IV contraction |
| ATM options | Highest vega (most sensitive to IV changes) |
| Long-dated options (LEAPS) | Very high vega |
| Short-dated options (weeklies) | Lower vega |

### IV Expansion vs. IV Crush

**IV Expansion** (IV rising) — happens when uncertainty increases (approaching earnings, macro events, geopolitical risk):
- Long options gain value even if the stock doesn't move
- The increase in uncertainty is priced into the option premium
- Strategy: Buy options before IV expansion, sell after

**IV Crush** — happens immediately after a binary event (earnings announcement, Fed meeting, major news):
- IV drops 30–60% in minutes
- Long options lose significant value even if the stock moves in your direction
- The "move" was already priced in. If the actual move is smaller than what IV implied, long options lose money.

**The earnings trap:** Many traders buy calls before earnings expecting a pop, get the pop, but still lose money because IV crush more than offsets the gain from delta. This is a vega loss overwhelming a delta gain.

**Solution:** 
- If you want directional exposure through earnings: use a debit spread (long and short options, vega partially offset)
- If you want to profit from IV crush: sell premium before earnings (strangle, condor) in an underlying you believe won't move dramatically

### Vega Math

```
Vega calculation:
  Option has vega of 0.20 and IV is currently 30%
  
  If IV rises from 30% to 31%: option gains $0.20 (per share = $20/contract)
  If IV rises from 30% to 35%: option gains $1.00 (5 points × $0.20)
  If IV falls from 30% to 25%: option loses $1.00 (5 points × $0.20)
  
  Post-earnings IV crush from 55% → 30% = 25-point drop
  Option loses 25 × 0.20 = $5.00 per share = $500 per contract from vega alone
```

---

## Rho — Interest Rate Sensitivity

### What It Measures
How much the option's price changes when risk-free interest rates change by 1%.

### Practical Relevance

- **Short-dated options (< 90 DTE):** Rho is negligible. Not worth tracking.
- **LEAPS (1–2+ years):** Rho becomes meaningful.
  - Rising interest rates → calls gain value, puts lose value
  - Falling interest rates → calls lose value, puts gain value
  - Rationale: higher rates increase the cost of carrying stock, which benefits call holders (can delay buying) and hurts put holders

**2026 note:** With rates elevated versus the 2010–2021 zero-rate environment, LEAPS are meaningfully more valuable for calls and less valuable for puts than in prior rate regimes. Factor this in when using LEAPS as long-term directional positions.

---

## Portfolio Greeks Dashboard

Update this at the start of every trading session when holding options positions.

```
═══════════════════════════════════════════════════════
ORACLE PORTFOLIO GREEKS — [DATE]
═══════════════════════════════════════════════════════

POSITIONS:
  [Ticker] [Strategy] [Expiry] — Δ: ____ Γ: ____ Θ: ____ V: ____
  [Ticker] [Strategy] [Expiry] — Δ: ____ Γ: ____ Θ: ____ V: ____
  [Ticker] [Strategy] [Expiry] — Δ: ____ Γ: ____ Θ: ____ V: ____

AGGREGATE:
  Net Delta:  ____  → SPY moves $1: portfolio moves $____
  Net Gamma:  ____  → (+) = benefit from big moves | (-) = risk from big moves
  Net Theta:  ____  → I gain/lose $____ per day from time decay
  Net Vega:   ____  → If IV rises 1 point across holdings: $____ gain/loss

EXPOSURE SUMMARY:
  Directional bias: Bullish / Bearish / Neutral
  Volatility bias: Long vol (benefiting from expansion) / Short vol (collecting IV)
  Time bias: Theta positive (want slow time) / Theta negative (need movement now)

RISK FLAGS:
  [ ] Any position within 7 DTE with gamma risk?
  [ ] Any position approaching earnings with unchecked vega risk?
  [ ] Net delta too large relative to account size?
  [ ] Any uncovered short positions (undefined risk)?

TODAY'S GREEK TARGETS:
  Target delta range: ______ to ______
  Max theta negative per day: $(______)
  Max vega exposure (1 IV point): $(______)
═══════════════════════════════════════════════════════
```

---

## Greeks by Strategy — Complete Reference

| Strategy | Delta | Gamma | Theta | Vega | Best Conditions |
|----------|-------|-------|-------|------|----------------|
| Long call | + | + | - | + | Bullish, low IV (buy cheap), expecting big move |
| Long put | - | + | - | + | Bearish, low IV, expecting big move |
| Bull call spread | + | + (small) | - (small) | + (small) | Bullish, moderate conviction, high IV (spread cheaper) |
| Bear put spread | - | + (small) | - (small) | + (small) | Bearish, moderate conviction, high IV |
| Iron condor | ~0 | - | + | - | Range-bound, high IV, no expected catalyst |
| Iron butterfly | ~0 | - (large) | + (large) | - (large) | Very range-bound, very high IV |
| Long straddle | ~0 | + | - | + | Low IV, big move expected (direction unknown) |
| Long strangle | ~0 | + | - | + | Low IV, big move expected (wider range than straddle) |
| Short strangle | ~0 | - | + | - | High IV, underlying stable, no catalyst |
| Covered call | + (reduced) | - (small) | + | - (small) | Mildly bullish, own shares, want income |
| Cash-secured put (CSP) | + | - (small) | + | - | Bullish at lower price, want to potentially own shares |
| Wheel (CSP → Covered Call) | + | - (small) | + | - | Moderately bullish, income-focused, high IV |
| LEAPS call (1+ year) | + (high) | + (small) | - (small) | + (high) | Strongly bullish long-term, want leverage without short-term risk |
| Calendar spread | ~0 | - (front) / + (back) | + | + | Flat short-term, expecting IV expansion or stock stability |
| Diagonal spread | + or - | Mixed | + | + (net) | Directional with time hedge |

---

## Greeks-Based Position Sizing

Use the Greeks to size positions intelligently, not just dollar amount.

### Delta-Based Sizing

```
Question: How much directional exposure do I want?
  
  Example: I want to trade like I own $10,000 of AAPL at $185
  Equivalent delta = $10,000 ÷ $185 = 54 delta units
  
  If I buy an ATM call (delta 0.50):
  54 delta ÷ 0.50 per share ÷ 100 shares per contract = 1.08 contracts
  → Buy 1 contract (gives ~50 delta units of equivalent exposure)
  
  This gives the same directional exposure as owning $9,250 of AAPL
  at a fraction of the capital outlay
```

### Theta Budget

If you're holding long options, establish a daily theta budget:

```
Max acceptable daily theta decay = 0.5% of account per day
  On $50,000 account: max theta = -$250/day
  On $100,000 account: max theta = -$500/day
  
  If net theta exceeds budget: close or convert some positions to reduce decay
```

### Vega Risk Limit

```
Max acceptable vega exposure (per 1-point IV move) = 1% of account
  On $50,000 account: max vega = ±$500 per IV point
  On $100,000 account: max vega = ±$1,000 per IV point
  
  If a portfolio-wide IV drop of 5 points could exceed this × 5: reduce vega
```

---

## Greek Interactions — How They Work Together

### The Theta-Gamma Trade-Off

Long options: You pay theta every day to own gamma. The more gamma (larger position size, closer to ATM, shorter dated), the more theta you pay. This is the core trade-off of long options: **you're renting explosive directional leverage, and the rent is theta.**

Short options: You collect theta every day but face gamma risk. If the stock moves sharply, your gamma works against you and losses can exceed all the theta collected. This is the core trade-off of short premium: **you're collecting rent but providing insurance.**

### The Vega-Theta Environment Signal

| IV Environment | Prefer Long Options? | Prefer Short Options? |
|---------------|---------------------|----------------------|
| Low IV (IV rank < 20) | Yes — options are cheap, buy premium | No — you'd be selling cheap premium |
| High IV (IV rank > 50) | No — options are expensive; IV crush risk | Yes — collect rich premium |
| Rising IV trend | Yes — vega positive positions gaining | No — vega negative positions losing |
| Post-event IV crush | No (already happened) | Yes — if IV remains elevated after first crush |

**IV Rank (IVR):** Measures where current IV sits relative to its 52-week range. IVR of 80 = IV is in the top 20% of its yearly range — selling premium is well-compensated. IVR of 10 = IV is cheap — buying premium is preferred.

---

## Quick Greeks Reference Card

```
FAST GREEKS LOOKUP

DELTA
  ATM = ~0.50  |  Deep ITM = 0.80-0.95  |  Far OTM = 0.05-0.20
  Portfolio: Net Δ × $1 move = my P&L
  Short puts and covered calls: positive delta exposure

GAMMA
  Highest: ATM options near expiration
  Long gamma = big moves help you
  Short gamma = big moves hurt you (especially near expiry)

THETA
  ATM options decay fastest
  Decay accelerates: 60 DTE → 30 DTE → 7 DTE (exponential)
  Long options bleed daily | Short options collect daily
  Rough rule: ATM weekly loses ~20% of extrinsic value per day in final week

VEGA
  All long options: positive vega (IV up = gain, IV down = lose)
  All short options: negative vega (IV up = lose, IV down = gain)
  Earnings: IV typically spikes before, crushes after
  Buy low IV | Sell high IV

POSITION CHECKLIST
  [ ] What is my net delta? (directional risk)
  [ ] Is gamma working for or against me?
  [ ] What is my daily theta? (time cost or income)
  [ ] What happens if IV moves 5 points? (vega stress test)
  [ ] Is there an earnings/event within my expiry? (vega trap check)
```
