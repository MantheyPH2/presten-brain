# ORACLE's Daily Options Routine

> Every day. No exceptions. The routine is the edge. Professionals have process; amateurs have impulses.

---

## Pre-Market (5:00 - 6:30 AM PT)

### IV Check on All Positions (5:00 - 5:30 AM)

Open your portfolio and for every active options position:

1. **Check IV rank and IV percentile** — has IV changed significantly overnight?
   - If IV spiked on a short premium position: assess whether to close early (the position is now more expensive to buy back)
   - If IV dropped on a long premium position: assess whether the thesis still justifies holding
   - Flag any position where IV rank crossed above 60 or below 20 overnight

2. **Check each position's Greeks as of the prior close:**
   - Delta: has directional exposure shifted? A 0.30 delta position may have moved to 0.50 delta if the stock moved toward your strike
   - Theta: how much did you earn/lose overnight from time decay? Running total for the week
   - Gamma: are any near-expiration positions developing dangerous gamma? (>7 DTE = manageable, <3 DTE = close or roll)
   - Vega: with the IV changes above, estimate your overnight P&L from vega alone

3. **Quick portfolio Greek snapshot:**
   - Net delta (beta-weighted to SPY)
   - Net theta (daily portfolio decay earnings)
   - Net vega (portfolio sensitivity to vol changes)
   - Are you within your target ranges? (Write your targets here: ______)

### News and Event Scan (5:30 - 6:00 AM)

Check for anything that affects your positions:

- **Earnings**: any holdings reporting today or this week? Pre/post market?
- **Economic data**: CPI, PPI, PCE, jobs report, FOMC? These move the entire market and IV levels
- **Geopolitical**: tariff announcements, geopolitical events, regulatory actions
- **Sector news**: FDA decisions, antitrust rulings, product launches affecting your positions
- **Futures**: where are S&P and Nasdaq futures? Significant gap up/down?
- **VIX futures**: is VIX up or down pre-market? This signals what IV will do at open.

Sources:
- Bloomberg or CNBC pre-market for headlines
- Earnings Whispers for the day's earnings calendar
- Economic calendar (Forex Factory or Trading Economics)
- Futures: /ES and /NQ on ThinkorSwim or Investing.com

### Plan the Day (6:00 - 6:30 AM)

Before the market opens, write down:

1. **Adjustments needed today:**
   - Any position hitting a profit target (50% of max profit for credit trades)?
   - Any position hitting a loss trigger (2x credit received)?
   - Any position that needs to be rolled (approaching expiration, < 21 DTE)?
   - Any position where Greeks have drifted outside tolerance?

2. **New trades planned:**
   - What setups from the playbook are active today?
   - What is the IV environment right now? (Favor selling if IV rank > 40, buying if < 25)
   - Any earnings plays to enter today?

3. **What you will NOT do today:**
   - If it is not on this list, you don't trade it. Period.
   - No FOMO trades. No revenge trades. No "I saw it on Twitter" trades.

---

## Market Open (6:30 - 7:00 AM PT)

### First 30 Minutes: Observe, Don't Trade

The open is chaos. Market makers are adjusting quotes, overnight orders are being filled, and spreads are at their widest. Your job for the first 30 minutes:

1. **Watch, don't click.** Observe how your positions react to the opening prices. Note any large gaps in the underlying stocks.

2. **Check opening IV levels vs yesterday:**
   - Did IV spike at open? (Common on gap-down days or pre-event days)
   - Did IV drop? (Common day-after-earnings, post-FOMC)
   - Compare the opening straddle price to yesterday's close for your key positions

3. **Look for mispriced options:**
   - IV rank extreme readings (>80 or <15) on your watchlist
   - Unusually wide bid-ask spreads that will tighten (potential opportunity once they normalize)
   - Opening prints that seem disconnected from fair value

4. **Confirm or revise your plan:**
   - Does the pre-market plan still make sense given the open?
   - Any new information that changes a thesis?
   - Adjust limit order prices based on actual opening levels

5. **Exception to the "don't trade" rule:** if a position hits a hard stop (e.g., short strike breached, max loss reached), close it immediately. Risk management doesn't wait.

---

## Active Trading (7:00 AM - 12:00 PM PT)

### Execute Planned Trades (7:00 - 8:00 AM)

This is your primary execution window. The 10:00-11:00 AM ET window (7:00-8:00 AM PT) has the best combination of tight spreads and stable IV.

**For each planned trade:**
1. Set up the order as a single spread (never leg in separately unless you have a specific reason)
2. Place a limit order at the mid price
3. Wait 30-60 seconds
4. If not filled, adjust by $0.01-$0.02 toward the natural price
5. Never use market orders on options. Ever.
6. Confirm fills and record entry Greeks in your journal

**Adjustments:**
- Close positions at 50% profit target using limit GTC orders (set these when you open the position)
- Roll positions that need rolling (close current, open new in a single order if your platform supports it)
- Close losers that hit your stop level

### Monitor Position Greeks (8:00 AM - 12:00 PM)

Check in every 60-90 minutes during this window:
- Has any position's delta moved significantly? (Stock approaching your strikes)
- Is portfolio net delta still within range?
- Any unusual IV movement on your positions?
- Check the SPY/QQQ for trend — is the market trending or range-bound?

**Trigger-based actions:**

| Trigger | Action |
|---------|--------|
| Credit position at 50% max profit | Close immediately |
| Credit position at 200% of credit received (loss) | Close immediately |
| Position < 21 DTE and not near max profit | Roll or close |
| Short strike breached | Close or roll for credit |
| Portfolio net delta exceeds +/- 800 SPY-weighted | Add hedge or reduce positions |
| VIX spikes > 5 points intraday | Review all short vega positions |
| IV rank on a holding drops below 20 | Close short premium positions (edge is gone) |

### Scan for New Setups (10:00 AM - 12:00 PM PT)

Use this window to scan for tomorrow's or next week's trades:
- Screen for high IV rank stocks (> 50) for potential premium selling
- Screen for low IV rank stocks (< 25) with upcoming catalysts for potential premium buying
- Check the earnings calendar for next week
- Review the playbook setups and see which have active signals today
- Add candidates to your watchlist with notes on which setup applies

**Screener criteria:**
- IV rank > 50 + liquid options (>1000 OI on ATM strikes) + 30-45 DTE expiration available = credit spread candidates
- IV rank < 25 + upcoming catalyst within 30 days + liquid options = debit spread or straddle candidates
- Stock at 52-week high/low + IV rank < 30 = LEAPS candidate (mean reversion)

---

## Market Close (12:00 - 1:00 PM PT)

### End-of-Day Review (12:00 - 12:30 PM)

1. **Review all positions one final time:**
   - Any positions expiring this week? Plan for Friday management (close by Thursday unless you want assignment)
   - Any positions that need overnight attention? (earnings after close, economic data pre-market tomorrow)
   - Check closing IV levels and compare to your opening snapshot

2. **Calculate end-of-day portfolio Greeks:**
   - Net delta (beta-weighted to SPY): _____
   - Net theta (daily): _____
   - Net vega: _____
   - Compare to your morning snapshot. What changed and why?

3. **Plan overnight exposure:**
   - Are you comfortable holding current positions overnight?
   - Any binary events tomorrow pre-market? (Economic data, earnings)
   - If you are concerned about overnight risk, consider closing or hedging positions before the close
   - Don't hold short premium positions through a binary event unless that is your explicit strategy

### Trade Log (12:30 - 1:00 PM)

Log every trade from today. For each trade:

```
Date:
Ticker:
Strategy:
Entry price:
Entry Greeks (delta, theta, vega):
IV rank at entry:
Thesis (1-2 sentences):
Planned exit: (profit target / stop loss / time-based)
Actual exit price (if closed today):
P&L:
Lessons:
```

Also log:
- Trades you DIDN'T take and why (this builds discipline awareness)
- Mistakes made today (be specific and honest)
- Emotional state during trading (calm? anxious? frustrated? overconfident?)

---

## After Hours (Evening)

### Analyze the Day's Trades (30 minutes)

Review your trade log from a higher level:
- How many trades today? (If > 5, you may be overtrading)
- Win rate this week (rolling)
- Average P&L per trade this week
- Are you following your rules? Check the plan you wrote this morning against what you actually did.
- Any patterns in your mistakes? (Same type of loss, same emotional trigger, same time of day)

### Study One New Concept (30 minutes)

Pick ONE thing to learn each day. Not five things superficially, one thing deeply.

Examples:
- Monday: study skew on a specific stock and understand why it's shaped that way
- Tuesday: analyze a historical earnings trade (did the expected move predict well?)
- Wednesday: read one chapter of Natenberg or Sinclair
- Thursday: study a strategy you haven't used yet (paper trade it mentally)
- Friday: review the week's vol surface changes and what drove them

Track what you studied in a running log. After 12 weeks you will have studied 60 distinct topics.

### Prepare Tomorrow's Watchlist (15 minutes)

Before bed (or whenever you do this):
1. Check futures and any after-hours earnings results
2. Review tomorrow's economic calendar
3. Identify 3-5 stocks for potential trades tomorrow
4. For each: note IV rank, recent price action, any catalysts, which playbook setup applies
5. Draft tomorrow's plan (adjustments needed, new trades, what to avoid)

Write it down. Tomorrow morning you refine it, not create it from scratch.

---

## Weekly Additions

### Friday End-of-Week

- Calculate weekly P&L (options only)
- Calculate portfolio Greeks trend (how did your aggregate deltas/theta/vega change week over week)
- Review all open positions: any that should be closed before the weekend?
- Close or roll any positions expiring next week that you don't want to manage through expiration
- Set next week's targets: # of new trades, target net theta, positions to adjust

### Sunday Evening (30 minutes)

- Review the upcoming week's earnings calendar
- Check VIX and IV rank levels across your watchlist
- Read the economic calendar for the week
- Draft the week's trading plan: which setups to focus on, which positions to manage
- Set specific goals: "this week I will practice 2 calendar spreads in paper" or "this week I will close all 50% winners within 1 hour of hitting target"

---

## The Rules That Never Break

1. **No trades before 7:00 AM PT** (10:00 AM ET) unless closing a risk position
2. **Every trade has a written thesis** before you click submit
3. **50% profit target on credit trades** — no exceptions, no "let it ride"
4. **2x credit received as max loss** on credit trades — no hoping for recovery
5. **No more than 5% of portfolio in a single trade**
6. **Check IV rank before every trade** — it determines whether you buy or sell premium
7. **Log every trade** the same day — a trade without a journal entry didn't happen
8. **No trading on tilt** — if you just took a loss and feel frustrated, close the platform for 30 minutes minimum
9. **Weekly Greek review** — if you don't know your portfolio Greeks, you don't know your risk
10. **One day off per week** — no charts, no scans, no trades. Your brain needs to reset.
