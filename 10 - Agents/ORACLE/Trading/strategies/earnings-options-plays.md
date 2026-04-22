# Earnings Options Plays

## The Earnings Landscape

Earnings announcements create the most predictable volatility events in the market. Implied volatility (IV) rises into earnings and crashes after (IV crush). This creates both opportunities and traps.

---

## Understanding IV Crush

**What it is**: IV builds up before earnings as uncertainty increases. After the announcement, uncertainty disappears and IV collapses -- often 40-70% overnight.

**Why it matters**: Even if you're directionally correct, IV crush can destroy your long option's value. You buy a call, the stock goes up 3%, but your call LOSES money because IV dropped 50%.

**The math**: A stock at $100 with pre-earnings IV of 80%. You buy a $105 call for $4.00. Earnings come out, stock moves to $103 (+3%). IV crushes to 30%. Your call is now worth $1.50. You were RIGHT on direction and still lost 62.5%.

---

## Pre-Earnings Strategies (Before the Announcement)

### Strategy 1: Pre-Earnings IV Run-Up (Long Straddle/Strangle)
**Thesis**: Buy volatility early (2-3 weeks before earnings) when IV is still low, sell it before the announcement when IV has inflated.

**Straddle**: Buy ATM call + ATM put, same expiration.
**Strangle**: Buy OTM call + OTM put, same expiration (cheaper, needs bigger move).

**Entry**: 2-3 weeks before earnings when IV is below its pre-earnings average
**Exit**: 1-2 days BEFORE the earnings announcement (sell the inflated IV, avoid the crush)

**Key**: You're not betting on the earnings result. You're betting that IV will rise into the event. This is a pure volatility trade.

**Best candidates**: Stocks with a history of IV rising significantly into earnings (check historical IV patterns).

### Strategy 2: Directional Pre-Earnings Trade
If you have a strong directional thesis (based on flow, fundamentals, or sector performance):
- Buy a debit spread (bull call spread or bear put spread) 2-3 weeks out
- Spreads are less affected by IV crush because both legs are affected
- Exit before earnings or hold through with defined risk

---

## Post-Earnings Strategies (After the Announcement)

### Strategy 3: Iron Condor (Sell the Crush)
**Thesis**: The stock won't move as much as the options market implies. Profit from IV collapse.

**Setup**: Sell an iron condor (short strangle inside a long strangle) right before earnings close.
- Put the short strikes at 1 standard deviation (expected move)
- Wide enough to survive the actual move

**Best for**: Large-cap, stable companies with a history of modest earnings reactions (AAPL, MSFT, JNJ).

**Risk**: If the stock gaps beyond your strikes, you take the max loss. This happens ~30% of the time.

### Strategy 4: Earnings Gap & Go (Post-Earnings Continuation)
**Thesis**: Big earnings gaps often continue for 2-5 days as institutions build/exit positions.

**Setup**:
1. Stock gaps significantly on earnings (> 5%)
2. Wait for the first pullback (usually 1-3 days after)
3. Enter a debit spread or long option in the gap direction
4. IV is already crushed, so options are cheap
5. Ride the continuation move

**Best for**: Stocks that gap with massive volume and institutional follow-through.

### Strategy 5: Earnings Put Sell (Post-Crush Premium Collection)
**Thesis**: After a stock drops on earnings, IV is still elevated. Sell puts on the dip to collect inflated premium.

**Setup**:
1. Stock drops 5-10% on earnings
2. If the fundamentals are still solid, sell cash-secured puts at a price you'd want to buy
3. IV is still elevated post-drop, so premiums are rich
4. Collect premium as IV continues to normalize

---

## When Each Strategy Works Best

| Strategy | Best When | Worst When |
|----------|-----------|------------|
| Pre-Earnings Straddle/Strangle | IV historically rises significantly pre-earnings | IV is already elevated early |
| Iron Condor | Stable stock, history of small moves | Stock has history of huge surprises |
| Gap & Go | Strong gap with massive volume | Gap on low volume or contradictory fundamentals |
| Post-Earnings Put Sell | Quality stock drops on soft earnings | Stock drops on structural problems |

---

## Critical Rules for Earnings Options

1. **Never buy naked calls/puts through earnings** unless you're prepared to lose the entire premium (IV crush)
2. **Spreads beat single legs** for through-earnings plays (IV crush affects both legs, partially canceling)
3. **Position size small**: Earnings are binary events. Risk 0.5-1% max.
4. **Know the expected move**: Options market tells you what it expects. If actual < expected, premium sellers win. If actual > expected, premium buyers win.
5. **Check historical earnings moves**: If a stock typically moves 5% on earnings and the options market implies 8%, that's an edge for sellers. If the market implies 3%, that's an edge for buyers.

---

## Calculating Expected Move

**Formula**: Straddle price / stock price = expected move percentage

Example: AAPL at $180. ATM straddle (nearest weekly) costs $9.00.
Expected move = $9 / $180 = 5%
Expected range: $171 - $189

If you think the actual move will be smaller: sell premium (iron condor, short straddle).
If you think the actual move will be larger: buy premium (long straddle/strangle).

---

## ORACLE's Verdict
- **Edge:** Real (specific strategies only)
- **Difficulty:** Medium-Hard
- **Best for:** Swing trading around catalysts
- **Win rate (estimated):** 65-75% for iron condors on stable stocks; 50-55% for directional plays
- **Risk/Reward:** Variable -- iron condors have high win rate but risk/reward is against you; directional plays have lower win rate but better R:R
- **Will I use this?** Yes -- selectively
- **Why:** Earnings create the most predictable volatility events in the market. The pre-earnings IV run-up strategy (buy early, sell before announcement) is genuinely smart because you avoid IV crush entirely. Iron condors on stable large-caps have a statistical edge because the market tends to overestimate earnings moves on these names. I'll avoid naked long options through earnings (IV crush is too punishing) and focus on spreads and volatility plays. The post-earnings gap-and-go is also worth trading because IV is already crushed and options are cheap for the continuation move.
