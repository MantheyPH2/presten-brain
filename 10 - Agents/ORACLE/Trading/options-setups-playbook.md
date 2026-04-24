# ORACLE's Options Setups Playbook

> Seven setups. These are the only plays you need. Master them, repeat them, refine them. Every trade you make should map to one of these setups. If it doesn't, you're freelancing — and freelancing in options is how you lose money.

---

## Setup 1: Earnings IV Crush Capture

### The Edge
Implied volatility inflates before earnings to price in the uncertainty of the announcement. After the announcement, uncertainty resolves and IV collapses — often 30-50% overnight. The market systematically overprices earnings moves roughly 70% of the time. You sell that overpriced insurance and collect when it crashes.

### Entry Criteria
- **Timing**: 1-3 days before earnings announcement (after IV has already ramped)
- **IV rank**: > 70 (options must be significantly more expensive than usual)
- **Stock**: mega-cap or large-cap with liquid options (AAPL, MSFT, GOOGL, META, AMZN, NVDA, TSLA, NFLX, AMD)
- **Options liquidity**: ATM bid-ask spread < $0.15, OI > 2,000 on strikes you're using
- **NOT**: don't use this on small-caps, biotech, or stocks facing existential binary events (FDA approvals, acquisition votes)

### Exact Structure

**Iron Condor (primary):**
- Sell the put at ~10% below current price (or 1x expected move below)
- Buy the put 5 points below that
- Sell the call at ~10% above current price (or 1x expected move above)
- Buy the call 5 points above that
- Expiration: the weekly that expires soonest after earnings (usually 2-5 days out)

**Short Strangle (if you have the margin and experience):**
- Sell the put at 1.2x expected move below current price
- Sell the call at 1.2x expected move above current price
- Wider strikes = higher probability of profit but less premium

### Position Sizing
- Max loss on iron condor = width of widest spread minus total credit received
- Risk no more than 1-2% of portfolio on any single earnings trade
- Example: $100,000 portfolio, max risk per trade = $1,000-$2,000
- If the iron condor has $500 max loss per contract, trade 2-4 contracts max

### Target Profit and Exits
- **Target**: close the morning after earnings for 50-70% of max profit
- Don't wait for full max profit — the stock can keep moving post-earnings
- Place a GTC limit order to close at 50% of max profit before earnings even happen
- **Max loss**: if the stock gaps beyond your short strike by more than the spread width, you hit max loss. Accept it. This is priced into the 70% win rate.
- **Never hold through a second trading day** after earnings. Take whatever profit or loss exists the morning after.

### When to Adjust
- If the stock gaps inside your iron condor (between your short strikes) but closer to one side: do nothing. This is normal.
- If the stock gaps outside your short strike but inside your long strike: close immediately at open. Don't wait for recovery.
- If the stock gaps beyond your entire spread: max loss is hit. Close at open and move on.
- No adjustments needed in most cases — this is a set-it-and-forget-it overnight trade.

### Real Example with Numbers

**NVDA earnings, stock at $850:**
- Expected move (from ATM straddle): $68 or ~8%
- IV rank: 82
- Iron condor:
  - Sell $780 put / Buy $770 put ($10 wide)
  - Sell $920 call / Buy $930 call ($10 wide)
  - Total credit received: $3.20 ($320 per contract)
  - Max loss: $10 - $3.20 = $6.80 ($680 per contract)
  - Probability of profit: ~72% (both short strikes outside expected move)
  - Risk/reward: risk $680 to make $320

**After earnings:** NVDA reports, gaps +5% to $892.50. Both short strikes are safe.
- IV crushes from 65% to 32%
- Iron condor can be closed for $0.80 ($80)
- Profit: $320 - $80 = $240 per contract (75% of max profit)
- Close at open. Don't hold.

---

## Setup 2: Momentum Debit Spread

### The Edge
When a stock breaks a key technical level with volume and conviction, the follow-through tends to continue for 5-15 days. By using a debit spread instead of buying naked options, you reduce the impact of theta decay and lower your cost. The key is to enter when IV is low — you get cheap options and any subsequent move increases both the intrinsic value AND IV (boosting your position with vega).

### Entry Criteria
- **Price action**: stock breaks above a 20-day high, or below a 20-day low, with volume 1.5x+ the 20-day average
- **IV rank**: < 40 (options are cheap — you're buying)
- **Trend**: 20-day EMA above 50-day EMA for bullish (below for bearish)
- **Catalyst**: ideally a fundamental reason for the breakout (earnings, upgrade, product launch, sector rotation)
- **NOT**: don't chase extended moves (stock already up 10%+ in a week), don't use on low-liquidity options

### Exact Structure

**Bull Call Spread (bullish breakout):**
- Buy the call at the first ITM or ATM strike
- Sell the call 5-10 points OTM (depending on stock price — aim for the spread to be ~5% of stock price wide)
- Expiration: 45-60 DTE (give the thesis time to work)

**Bear Put Spread (bearish breakdown):**
- Buy the put at the first ITM or ATM strike
- Sell the put 5-10 points OTM
- Same DTE

### Position Sizing
- Max loss = debit paid
- Risk no more than 2% of portfolio per trade
- Example: $100,000 portfolio, max risk = $2,000. Debit spread costs $3.50 per contract ($350). Trade 5-6 contracts max.

### Target Profit and Exits
- **Target**: 50-75% of max profit, or if the stock reaches the short strike before expiration
- **Time stop**: if the trade hasn't moved in your direction within 10 trading days, close for whatever it's worth. The thesis was wrong.
- **Stop loss**: close if the spread loses 50% of its value (debit paid $3.50, close if it drops to $1.75)
- **Trail**: if the spread is at 60% profit but the stock is still moving, trail the stop to 40% profit and let it run

### When to Adjust
- If the stock reverses back below the breakout level: close immediately. The breakout failed.
- If the stock consolidates near the breakout: hold, but tighten your time stop to 5 more trading days.
- If the stock rockets through your short strike: consider closing the short leg and letting the long call run (but only if you have a very strong directional thesis).

### Real Example with Numbers

**AMD breaks above $160 resistance on volume:**
- AMD at $163, just broke above $160 resistance that held for 3 weeks
- Volume: 85M shares (avg is 55M) — 1.55x average
- IV rank: 28 (options are cheap)
- 20 EMA > 50 EMA (uptrend confirmed)
- Catalyst: data center GPU demand growing, sector rotation into semis

**Bull call spread:**
- Buy $160 call, 50 DTE: $9.50
- Sell $175 call, 50 DTE: $4.20
- Net debit: $5.30 ($530 per contract)
- Max profit: $15 - $5.30 = $9.70 ($970 per contract)
- Risk/reward: risk $530 to make $970 (1:1.83)
- Break-even: $165.30

**Trade 5 contracts: total risk $2,650, max profit $4,850**

**Result:** AMD runs to $178 over 3 weeks. The $160/$175 spread is now deep ITM.
- Spread value: $13.80
- Profit: $13.80 - $5.30 = $8.50 per contract ($850)
- Total profit: $4,250 on $2,650 risk (160% return)
- Close at 87% of max profit. Don't wait for the last 13%.

---

## Setup 3: The Wheel on High-Premium Stocks

### The Edge
You get paid to wait to buy stocks you want to own. Then you get paid to wait to sell them. The cash-secured put gives you a lower cost basis than buying shares outright. The covered call generates income while you hold. Over time, the premium collected compounds into a significant discount on your shares. The edge is in selecting stocks with high IV (more premium) that you genuinely want to own.

### Entry Criteria
- **Stock selection**: you must WANT to own 100 shares at the put strike. This is not a pure income play — you WILL get assigned.
- **IV rank**: > 50 (enough premium to justify tying up capital)
- **Stock price**: allows 100 shares without concentrating your portfolio (for $100k portfolio, stocks $20-$80 ideally)
- **Liquidity**: bid-ask spread on ATM options < $0.10, OI > 1,000
- **Fundamentals**: growing revenue, reasonable valuation, secular tailwinds. You are owning this stock long-term.
- **Best candidates**: AAPL, AMD, SOFI, PLTR, MARA, SQ, F, BAC, INTC, HOOD

### Exact Structure

**Phase 1 — Sell Cash-Secured Put:**
- Strike: ~0.30 delta (about 70% probability OTM, typically 5-10% below current price)
- Expiration: 30-45 DTE
- Requires: enough cash to buy 100 shares at the strike price ($5,000 cash for a $50 strike)

**Phase 2 — If Assigned (you now own 100 shares):**
- Calculate your cost basis: strike price minus premium received from put
- Immediately sell a covered call
- Strike: at or above your cost basis, ~0.30 delta
- Expiration: 30-45 DTE

**Phase 3 — If Called Away:**
- You sold shares at the call strike plus you collected put premium plus call premium
- Return to Phase 1

### Position Sizing
- Each Wheel position requires cash equal to 100x the put strike
- Don't allocate more than 10-15% of portfolio to a single Wheel position
- Run 2-3 Wheels simultaneously on different stocks for diversification

### Target Profit and Exits
- **CSP phase**: close at 50% of premium received. Then sell a new CSP for the next cycle. More efficient than waiting for full expiration.
- **CC phase**: close at 50% of premium received. Sell another CC.
- **If stock drops significantly while assigned**: do NOT sell CCs below your cost basis (you'd lock in a loss if called away). Sell CCs AT your cost basis or wait for the stock to recover.
- **When to stop the Wheel**: if the fundamental thesis breaks (bad earnings, deteriorating business), sell the shares and stop. Don't Wheel into a black hole.

### When to Adjust
- **Stock drops after CSP sold**: if the stock drops 15%+ and your put is deep ITM, roll the put down and out (lower strike, later expiration) for a credit. Maximum 1 roll — if it drops further, accept assignment and switch to CC phase.
- **Stock rockets after CC sold**: roll the call up and out for a credit. Only if you can get a net credit. If not, let it get called away and start over — being called away is profitable by definition (you sold above cost basis).
- **IV drops significantly**: close the position early even at less than 50% profit. The remaining premium doesn't justify the capital tie-up.

### Real Example with Numbers: 3 Full Cycles

**Stock: SOFI at $12.50, IV rank 58**

**Cycle 1:**
- Sell $11.50 CSP, 35 DTE, for $0.55 ($55 per contract)
- Stock stays above $11.50. Close at 50% ($0.275) after 18 days.
- Profit: $27.50
- Sell new $11.50 CSP for next cycle...

**Stock drops. Assigned at $11.50. Cost basis: $11.50 - $0.55 = $10.95**

**Cycle 2:**
- Sell $12 CC, 30 DTE, for $0.50 ($50)
- Stock climbs to $12.80. Called away at $12.
- Profit on shares: $12.00 - $10.95 = $1.05 ($105)
- Plus CC premium: $50
- Total Cycle 2 profit: $155
- Return to CSPs

**Cycle 3:**
- Sell $12 CSP, 30 DTE, for $0.45 ($45)
- Close at 50% after 15 days: $22.50 profit

**Total 3-cycle P&L:**
- Cycle 1 premiums: $27.50
- Cycle 2 (shares + premium): $155
- Cycle 3 premiums: $22.50
- **Total: $205 profit on ~$1,150 capital deployed over ~3 months**
- **Annualized return: ~71%** (this is realistic for high-IV stocks, lower for blue chips)

---

## Setup 4: LEAPS + Short Call (Poor Man's Covered Call)

### The Edge
You capture 80% of the upside of owning shares for 20-25% of the capital. Meanwhile, you sell monthly calls against the LEAPS to generate theta income. The combination gives you leveraged upside with income — the best of both worlds. The edge is capital efficiency: freeing up 75-80% of your capital for other trades.

### Entry Criteria
- **Conviction**: strong 12-18 month bullish thesis on the stock
- **IV rank on LEAPS**: < 40 ideally (buy cheap LEAPS)
- **IV rank on monthly options**: > 30 (sell decent short-term premium)
- **Stock**: mega-cap or large-cap with liquid LEAPS (tight spreads even on 12-18 month options)
- **NOT**: don't use on volatile small-caps where the LEAPS can easily go to zero. Don't use when IV on the LEAPS is extremely high (you'll overpay for the long leg).

### Exact Structure

**Buy the LEAPS (long leg):**
- Deep ITM call, 0.80+ delta
- 12-18 months to expiration
- Typically 15-25% ITM
- This is your "shares replacement"

**Sell the monthly call (short leg):**
- 0.20-0.30 delta OTM
- 30-45 DTE
- This is your income stream
- Roll monthly: when it hits 50% profit or 21 DTE, close and sell a new one

### Position Sizing
- Each PMCC requires the LEAPS cost as capital (much less than 100 shares)
- Max loss = LEAPS cost minus any premium collected from short calls
- Don't allocate more than 8-10% of portfolio to a single PMCC position
- Advantage: you can run 4-5 PMCCs for the same capital as 1 covered call position

### Target Profit and Exits
- **Short call**: close at 50% of premium received each cycle. Sell a new one.
- **LEAPS**: hold for 6-12 months if thesis intact. Roll the LEAPS to a new expiration when it reaches 6 months remaining (avoids accelerated decay).
- **Full position close**: take profit when the LEAPS has doubled in value, or when the thesis is complete (stock hit your target).
- **Stop loss on LEAPS**: if the LEAPS drops to 50% of purchase price, reassess the thesis. Close if the thesis is broken.

### When to Adjust
- **Short call challenged (stock rises fast)**: roll the short call up and out for a credit. The LEAPS is gaining value too, so the overall position profits.
- **Stock drops moderately**: stop selling short calls temporarily. Let the LEAPS participate in any bounce without being capped. Resume selling calls after stabilization.
- **Stock drops significantly (LEAPS delta drops below 0.60)**: close everything or roll the LEAPS down to a lower strike (though this usually means taking a loss on the original LEAPS).
- **LEAPS approaches 6 months remaining**: roll to a new 12-18 month expiration at the same or similar strike.

### Real Example with Numbers

**MSFT at $420, bullish 12-month thesis (AI revenue acceleration):**

**Buy LEAPS:**
- Buy $350 call, 14 months out: $82.00 ($8,200 per contract)
- Delta: 0.82
- Alternative: buying 100 shares costs $42,000 — LEAPS saves $33,800 in capital (80% reduction)

**Sell monthly calls (cycle 1):**
- Sell $445 call, 35 DTE: $5.20 ($520)
- Delta: 0.25

**Month 1:** MSFT goes to $435. Short call expires worthless.
- LEAPS gained: ~$12.30 ($1,230) from delta
- Short call income: $520
- Month 1 total: +$1,750

**Sell next monthly call (cycle 2):**
- Sell $460 call, 30 DTE: $4.80 ($480)
- Close at 50% after 16 days: $240 profit

**Month 2:** MSFT goes to $445.
- LEAPS gained: ~$8.20 ($820)
- Short call income: $240
- Month 2 total: +$1,060

**Running total after 2 months:**
- LEAPS unrealized gain: ~$2,050
- Short call income: $760
- Total position value increase: $2,810 on $8,200 capital = 34% return in 2 months
- Annualized yield from short calls alone: ~$4,560 / $8,200 = 55%

---

## Setup 5: Event-Driven Straddle

### The Edge
Before major binary events (FOMC, CPI, geopolitical), the market often underprices the move 3-5 days in advance. IV builds gradually into the event, but the cheapest time to buy is before the buildup starts. You buy a straddle when options are cheap and sell it when IV has inflated — OR hold through the event if the straddle is cheap enough relative to the likely move.

### Entry Criteria
- **Event**: clear binary catalyst — FOMC decision, CPI/PCE release, election, major geopolitical event, FDA decision
- **Timing**: buy 3-5 trading days before the event (before IV fully prices in the event premium)
- **IV rank**: < 40 at time of purchase (the event premium hasn't built yet)
- **Expected move**: your estimate of the likely move must exceed the straddle cost for hold-through-event strategy
- **Stock/ETF**: SPY, QQQ, or individual stocks directly affected by the event

### Exact Structure

**Long Straddle:**
- Buy ATM call + ATM put
- Same strike, same expiration
- Expiration: the weekly that expires 1-3 days AFTER the event (gives time for follow-through)

**Long Strangle (cheaper alternative):**
- Buy OTM put (1-3% OTM) + OTM call (1-3% OTM)
- Cheaper entry but needs a bigger move to profit

### Position Sizing
- Max loss = total premium paid (both legs)
- Risk 1-2% of portfolio per event trade
- This is a lower win-rate, higher payoff strategy. Size accordingly.

### Target Profit and Exits

**Strategy A — Sell the IV pump (before the event):**
- Buy the straddle 5 days before the event
- Sell 1 day before the event when IV has peaked
- Target: 20-40% profit from IV expansion alone (even without the stock moving much)
- This is the lower-risk approach

**Strategy B — Hold through the event:**
- Hold the straddle through the announcement
- If the move exceeds the straddle cost, you profit
- Target: 50-100% profit on a large move
- Risk: IV crushes after the event AND the stock doesn't move enough. You can lose 40-60% of premium overnight.

**Stop loss**: if the straddle loses 30% of its value before the event (stock isn't moving AND IV isn't building), close. Something is wrong with the thesis.

### When to Adjust
- If the stock moves significantly in one direction before the event, one leg becomes very profitable while the other dies. Consider closing the winning leg to lock in profit and hold the losing leg as a cheap lottery ticket through the event.
- If IV builds faster than expected (straddle is up 30% in 2 days), consider taking profit early. Don't be greedy.

### Real Example with Numbers

**FOMC decision in 4 days, SPY at $540:**
- IV rank on SPY: 32 (event premium hasn't fully built)
- ATM straddle (weekly expiring 2 days after FOMC):
  - Buy $540 call: $5.80
  - Buy $540 put: $5.40
  - Total cost: $11.20 ($1,120 per contract)
  - Break-even: SPY above $551.20 or below $528.80 (about +/- 2.1%)

**Strategy A execution:**
- Buy straddle at $11.20 on Monday
- By Wednesday (day before FOMC), IV has risen. Straddle is now $13.50 even though SPY barely moved.
- Sell for $13.50. Profit: $2.30/contract ($230). Return: 20.5%.
- Low risk, captured pure IV expansion.

**Strategy B execution:**
- Hold through FOMC. Fed surprises hawkish. SPY drops to $528.
- Put is now worth $12.00, call is worth $0.40
- Straddle value: $12.40. Profit: $1.20/contract ($120). Return: 10.7%.
- But if SPY had dropped to $522: put worth $18.00, total $18.40, profit $720 (64% return).
- If SPY hadn't moved: IV crush kills both legs. Straddle worth ~$5.50, loss of $570 (51% loss).

Strategy A is the safer play. Strategy B is for when you believe the move will be large.

---

## Setup 6: Inflation Data Hedge Spread

### The Edge
CPI, PCE, and PPI releases move markets. If you are long equities (which ORACLE usually is), a hot inflation print can tank the market 1-3% in hours. Rather than selling your entire portfolio before the data, use a cheap put spread as defined-risk insurance. If inflation is hot and the market drops, the put spread profits and offsets your portfolio losses. If inflation is benign, you lose only the small debit paid.

### Entry Criteria
- **Event**: CPI, Core CPI, PCE, Core PCE, PPI release (check economic calendar)
- **Timing**: enter 1-2 days before the release
- **Portfolio context**: you have net long equity exposure that would suffer from a hot inflation print
- **Market positioning**: check SPY/QQQ IV around the data date — if IV is already elevated (rank > 50), the market is already hedged and the put spread is expensive. Works best when IV rank < 40.

### Exact Structure

**Bear Put Spread on SPY or QQQ:**
- Buy a put 1-2% OTM (for SPY at $540, buy the $530 put)
- Sell a put 3-4% OTM (sell the $520 put)
- Width: $10 ($1,000 per contract on SPY)
- Expiration: the weekly expiring the same day as the data release, or 1-2 days after

**Sizing as a hedge:**
- Calculate your portfolio's approximate loss if SPY drops 2%
- Size the put spread so its max profit roughly equals that loss
- Example: $100k portfolio, beta 1.1, 2% SPY drop = ~$2,200 loss
- Put spread max profit per contract: $10 width - $2.00 debit = $8.00 ($800)
- Need ~3 contracts to offset: 3 x $800 = $2,400 protection
- Cost: 3 x $2.00 = $6.00 ($600). That's 0.6% of portfolio for downside protection.

### Position Sizing
- Hedge cost should be 0.3-0.8% of portfolio value
- If the hedge costs more than 1%, options are too expensive — consider a narrower spread or skip this month
- This is insurance, not a profit play. Think of it as your portfolio's health insurance premium.

### Target Profit and Exits
- **If inflation is hot (market drops)**: close the put spread within 2 hours of the data release. The initial reaction is often the largest — don't wait for further downside.
- **If inflation is benign (market flat or up)**: the put spread expires worthless or close for whatever residual value. This is expected. You paid $600 for insurance you didn't need — that's a good outcome for your portfolio.
- **If data is mixed**: the put spread may be worth 30-50% of its value. Close and take what you get.

### When to Adjust
- No adjustments. This is a simple binary hedge.
- If you want more protection, add contracts. If you want less, remove them.
- If the market drops before the data release (positioning), your put spread gains value. Consider closing for profit and re-entering at lower strikes.

### Real Example: The Exact Play

**PCE data releasing Friday, April 25. ORACLE is long equities.**

- SPY at $540, QQQ at $460
- ORACLE's portfolio: $85,000, beta-weighted delta to SPY: +620 (equivalent to $33,500 in SPY exposure)
- A 2% SPY drop would cost approximately: 620 x $1.08 = ~$670 per 0.2% move, x10 = ~$6,700 on a 2% drop. Call it ~$1,700 on a realistic 0.5% hot-print sell-off.

**The hedge:**
- Buy QQQ $450 put, expiring April 25: $3.40
- Sell QQQ $440 put, expiring April 25: $2.10
- Net debit: $1.30 ($130 per contract)
- Max profit: $10 - $1.30 = $8.70 ($870 per contract)
- Trade 2 contracts: $260 cost, $1,740 max protection

**Scenario 1 — Hot PCE print:**
- Core PCE comes in at 0.4% MoM vs 0.3% expected
- QQQ drops 1.5% to $453
- Put spread still OTM but closer: worth ~$3.50. Profit: $2.20 x 2 = $440
- Portfolio loss from long positions: approximately $1,000
- Net loss: $560 instead of $1,000. The hedge cut the damage by 44%.

**Scenario 2 — In-line PCE:**
- Core PCE 0.3% as expected. QQQ flat to slightly up.
- Put spread expires worthless. Loss: $260.
- Portfolio: flat to up. Net result: portfolio gain minus $260 insurance cost.

**Scenario 3 — Cool PCE (surprise to downside):**
- Core PCE 0.2%. QQQ rips +1.5%.
- Put spread worthless. Loss: $260.
- Portfolio gains ~$1,000+. Net: strong day, insurance cost was trivial.

The $260 cost is 0.3% of the portfolio. Worth it for the peace of mind and the real protection on hot prints.

---

## Setup 7: Volatility Mean Reversion

### The Edge
The VIX has a long-term mean around 17-20. It spikes during fear events (sometimes to 30, 40, even 80) but it ALWAYS reverts to the mean. Always. The question is timing. When VIX spikes above 30, the statistical edge of selling VIX calls or buying VIX puts is enormous — but you need to be right about the timing (VIX can stay elevated for days or weeks) and survive the drawdown if it spikes further before reverting.

### Entry Criteria
- **VIX level**: > 28 for moderate signal, > 35 for strong signal
- **Context**: the VIX spike should be from a fear event (geopolitical shock, tariff announcement, credit scare, liquidity event), NOT from a fundamental regime change (early 2020 COVID was a regime change, not a tradeable spike)
- **VIX term structure**: inverted (near-term VIX futures > longer-term). This confirms panic and is the strongest mean-reversion signal.
- **Timing**: don't sell the first spike. Wait 1-2 days for the initial shock to settle. VIX often spikes, drops partially, then spikes again (double spike pattern). Enter after the second spike or after 2 consecutive down days in VIX.
- **NOT**: don't sell VIX at 22 hoping for 17. The risk/reward is poor. Wait for true spikes above 28-30.

### Exact Structure

**Option A — Sell VIX Call Spreads:**
- Sell VIX call at a strike near current level (if VIX at 32, sell the 32 call)
- Buy VIX call 5-10 points higher (buy the 40 call)
- Expiration: 30-45 days out (VIX reversion takes time)
- You profit as VIX drops and the call spread loses value
- Max profit = credit received. Max loss = width minus credit.

**Option B — Buy VIX Put Spreads:**
- Buy VIX put at current level or slightly below (VIX at 32, buy the 30 put)
- Sell VIX put 5-10 points lower (sell the 22 put)
- Expiration: 30-45 days
- You profit as VIX drops toward the put spread
- Max profit = width minus debit. Max loss = debit paid.

**Option C — SPY/QQQ Call Spreads (simpler):**
- When VIX is high, stocks are depressed. Sell SPY/QQQ put spreads or buy call spreads to bet on recovery.
- Less direct exposure to vol reversion but simpler to execute and manage.

### Position Sizing
- This is a high-conviction, moderate-frequency trade. Size 2-3% of portfolio.
- Account for the possibility that VIX goes to 40 or 50 before reverting. Your max loss must be survivable.
- Don't go all-in on the first entry. Scale in: 1/3 position at VIX 30, 1/3 at VIX 35, 1/3 at VIX 40.

### Target Profit and Exits
- **Target**: 50-60% of max profit on the spread
- **Time target**: VIX typically reverts to sub-25 within 10-30 trading days of a spike above 30. If it hasn't reverted in 30 days, reassess.
- **Stop loss**: if VIX goes 10+ points above your entry (e.g., you sold at VIX 32 and it hits 42), close or hedge. The spike may be turning into a sustained regime.
- **Scaling out**: close 1/3 of position when VIX drops to 25, another 1/3 at 22, final 1/3 at 19 (or when spread hits 50% profit).

### When to Adjust
- **VIX spikes higher after entry**: if you scaled in and have remaining capital allocated, add at the higher level. If fully sized, hold and manage risk. The thesis (mean reversion) is still valid — it just takes longer.
- **VIX drops quickly**: take profit aggressively. Fast VIX drops often reverse partially (dead cat bounce in fear). Close at least half the position on the initial drop.
- **Term structure normalizes** (no longer inverted): this is a strong signal that the fear event is passing. Take profit on remaining position.

### Real Example with Numbers

**Tariff panic pushes VIX to 34. Term structure inverted.**

Day 1: VIX spikes to 34. Watch. Don't trade.
Day 2: VIX drops to 30, then spikes back to 33. Double spike confirmed.
Day 3: VIX closes at 31. Enter position.

**Sell VIX call spread:**
- Sell VIX 31 call, 35 DTE: $4.80
- Buy VIX 40 call, 35 DTE: $2.10
- Net credit: $2.70 ($270 per contract)
- Max loss: $9 - $2.70 = $6.30 ($630 per contract)
- Trade 4 contracts: $1,080 credit, $2,520 max risk

**Week 1:** VIX drops to 26. Spread is now worth $1.60.
- Unrealized profit: $2.70 - $1.60 = $1.10 x 4 = $440 (41% of max profit)
- Close 2 contracts: lock in $220 profit

**Week 2:** VIX drops to 21. Spread is now worth $0.30.
- Remaining 2 contracts: profit = $2.40 x 2 = $480
- Close remaining position.

**Total profit: $220 + $480 = $700 on $2,520 max risk (28% return in 2 weeks)**

VIX reversion delivered as expected. The edge is real and repeatable — but only at extremes.

---

## Playbook Summary

| Setup | When | IV Environment | Frequency | Avg Win Rate | Risk per Trade |
|-------|------|----------------|-----------|-------------|---------------|
| 1. Earnings IV Crush | Pre-earnings, IV rank > 70 | Sell high IV | 4-8x per quarter | ~70% | 1-2% |
| 2. Momentum Debit Spread | Breakout + volume, IV rank < 40 | Buy low IV | 2-4x per month | ~55% | 2% |
| 3. The Wheel | Ongoing, IV rank > 50 | Sell high IV | Continuous | ~75% (per cycle) | 10-15% (capital tie-up) |
| 4. LEAPS + Short Call | 12-month thesis, LEAPS IV < 40 | Buy low/sell high | 2-3 positions | ~65% | 8-10% |
| 5. Event Straddle | 3-5 days before binary event | Buy low IV | 1-2x per month | ~45% | 1-2% |
| 6. Inflation Hedge | Pre-CPI/PCE/PPI | Buy (hedge) | Monthly | N/A (insurance) | 0.3-0.8% |
| 7. Vol Mean Reversion | VIX > 28-30 | Sell high vol | 2-4x per year | ~75% | 2-3% |

**The rules that apply to ALL setups:**
1. Check IV rank before entering. It determines whether you buy or sell premium.
2. Size so that max loss is 1-3% of portfolio (except Wheel, which is a capital allocation decision).
3. Use limit orders only. Never market orders.
4. Have exit criteria BEFORE you enter.
5. Log every trade with thesis, entry, exit, and lessons.
6. If a setup's entry criteria aren't met, don't force it. Wait.
