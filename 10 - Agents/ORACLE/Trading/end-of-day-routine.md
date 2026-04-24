---
type: trading-tool
name: End of Day Routine
last_updated: 2026-04-23
trigger: Every market day after 1:00 PM PT close
estimated_time: 45-60 minutes
---

# ORACLE End-of-Day Routine

Execute this routine EVERY trading day after market close (1:00 PM PT). No shortcuts. No skipping steps. This is what separates disciplined traders from gamblers. The post-market work is more important than the market hours work.

---

## Step 1: Log All Trades from Today
**Time: 5-10 minutes**

Open the trading journal (`trading-journal.md`) and log every trade taken today.

For each trade, record:
```
### Trade [#] — [TICKER] [LONG/SHORT]

- **Entry Time:** [HH:MM PT]
- **Entry Price:** [price]
- **Exit Time:** [HH:MM PT]
- **Exit Price:** [price]
- **Position Size:** [shares/contracts/tokens]
- **P&L:** [$ amount] ([% return])
- **Setup Type:** [ORB / VWAP bounce / Breakout / Mean reversion / Options / etc.]
- **Thesis:** [Why did you enter?]
- **What Happened:** [Brief narrative]
- **Followed the Plan:** [Yes / No / Partially]
- **Emotions During Trade:** [Calm / Anxious / FOMO / Revenge / Confident]
```

If no trades were taken, log that too: "No trades today. Reason: [no setups / discipline / studying]." Not trading IS a valid outcome.

---

## Step 2: Grade Each Trade
**Time: 5 minutes**

Grade each trade on PROCESS, not outcome. A losing trade with perfect process gets an A. A winning trade with terrible process gets an F.

| Grade | Criteria |
|-------|----------|
| **A** | Followed the plan exactly. Entry, stop, target all per pre-trade plan. Position size correct. Emotions managed. |
| **B** | Mostly followed the plan. Minor deviation (entered slightly early/late, adjusted stop once). Good discipline overall. |
| **C** | Plan was there but execution was sloppy. Moved stop, added to loser, entered without confirmation. Needs improvement. |
| **F** | No plan. Revenge trade. FOMO entry. Removed stop loss. Over-sized position. Traded during lunch chop. Chased. |

### Grading Rules
- A losing A-grade trade is worth more than a winning F-grade trade. The A-trade process will make money over 100 trades. The F-trade process will lose money over 100 trades.
- If you have more than 1 F-grade trade in a week, STOP TRADING LIVE. Go back to paper trading for 3 days minimum.
- Track your grade distribution monthly. Target: >50% A/B grades.

---

## Step 3: Update P&L on All Open Positions
**Time: 5 minutes**

Check current prices on all open positions (stocks, options, crypto) and update the P&L.

```
### Open Position P&L — [DATE]

| Ticker | Type | Entry | Current | P&L $ | P&L % | Stop | Target | Days Held |
|--------|------|-------|---------|-------|-------|------|--------|-----------|
| | | | | | | | | |

**Total Open P&L:** $[amount] ([%])
**Total Realized P&L Today:** $[amount]
**Running Monthly P&L:** $[amount]
```

### WebSearch Queries for Current Prices
```
"[TICKER] stock price after hours" site:finance.yahoo.com
"[TOKEN] price" site:coingecko.com
```

### Rules
- If any position is down >5% from entry, REVIEW the thesis. Is the stop still appropriate?
- If any position has hit 80% of target, consider taking partial profits
- If any position has been held >2 weeks with no movement, reassess — dead money is opportunity cost

---

## Step 4: Run the Portfolio Stress Test
**Time: 5 minutes**

Quick stress test across three scenarios. Reference `portfolio-stress-test.md` for the full methodology.

| Scenario | Impact on Portfolio |
|----------|-------------------|
| **SPY drops 3% tomorrow** | What happens to each position? Total portfolio drawdown? |
| **BTC drops 10% overnight** | What happens to crypto positions? COIN? Correlated assets? |
| **VIX spikes to 35** | Which positions benefit (puts, VIX calls)? Which suffer? |

### Rules
- If any single scenario causes >10% portfolio drawdown, you are OVER-CONCENTRATED. Reduce position sizes immediately.
- If the worst-case scenario (all three simultaneously) causes >20% drawdown, you are OVER-LEVERAGED. De-risk immediately.
- Always know your max loss before it happens. Surprises are for birthdays, not portfolios.

---

## Step 5: Check After-Hours Earnings Reactions
**Time: 5-10 minutes**

### WebSearch Queries
```
"after hours earnings today" site:benzinga.com
"earnings after close today" site:seekingalpha.com
"earnings whispers after hours" site:earningswhispers.com
```

### What to Look For
- **Earnings from your coverage universe:** If AAPL, MSFT, GOOGL, NVDA, etc. reported, immediately update the tracker file with results and after-hours price reaction
- **Sector read-throughs:** If a company in the same sector as your positions reported, what does it signal? (e.g., if INTC beats, is it good for AMD?)
- **Guidance changes:** Revenue guidance matters more than the earnings beat/miss. Forward guidance moves stocks more than backward-looking results
- **After-hours price reaction:** Log the AH move. If >5% move, add to tomorrow's HOT watchlist scan

### Actions
- Update relevant tracker files with earnings data
- If an HOT/WARM watchlist name reported: update the watchlist entry with results
- If a name gaps >5% AH on earnings: prepare a trade plan for tomorrow's open (gap fill or continuation?)

---

## Step 6: Update Tomorrow's HOT Watchlist
**Time: 5 minutes**

Review the HOT watchlist in `tiered-watchlist.md` and update for tomorrow.

### Checklist
- [ ] Remove any names where the setup expired today
- [ ] Remove any names where the trade was completed (win or loss)
- [ ] Promote any WARM names whose trigger fired today
- [ ] Add any new scanner hits from today that have immediate actionability
- [ ] Re-rank the list — #1 is tomorrow's highest-conviction trade
- [ ] Ensure every HOT name has updated entry/stop/target levels

### Rules
- Maximum 10 names. If full, remove one before adding one.
- Every name must have a trade plan for tomorrow. No "maybe" entries.

---

## Step 7: Identify Tomorrow's Potential Catalysts
**Time: 5 minutes**

### WebSearch Queries
```
"earnings tomorrow before open" site:earningswhispers.com
"economic calendar tomorrow" site:forexfactory.com
"fed speakers tomorrow" site:federalreserve.gov
"IPO calendar this week" site:nasdaq.com
"crypto events tomorrow" site:coinmarketcal.com
```

### Catalog Catalysts
```
### Tomorrow's Catalysts — [DATE]

**Before Market Open:**
- [Economic data: CPI/PPI/Jobs/GDP at 5:30 AM PT?]
- [Earnings before bell: Which companies?]
- [Fed speakers: Who and when?]

**During Market Hours:**
- [FOMC decision / minutes?]
- [Options expiration? (OpEx Fridays)]
- [Sector-specific events?]

**After Hours:**
- [Earnings after close: Which companies?]
- [Crypto events: Unlocks, upgrades, conferences?]

**Risk Events:**
- [Geopolitical: Anything brewing?]
- [Policy: Any government/regulatory announcements expected?]
```

---

## Step 8: Set Price Alerts on Key Levels
**Time: 3 minutes**

For every name on the HOT and WARM watchlists, ensure price alerts are set at:
1. Entry price (trigger to execute trade)
2. Stop loss level (trigger to exit)
3. Target price (trigger to take profits)
4. Key support levels (warning of breakdown)
5. Key resistance levels (warning of breakout)

### How to Set Alerts
- Use broker platform alerts for stocks/options
- Use TradingView alerts for crypto
- Use WebSearch to find current price distance from alert levels

### Rule
If you do not have alerts set, you are relying on memory and screen-watching. Both are unreliable. Alerts are non-negotiable.

---

## Step 9: Update Market Thesis if Anything Changed
**Time: 5 minutes**

Ask yourself these questions:
1. Did today's price action change my overall market thesis? (Bull/Bear/Neutral)
2. Did any sector rotation occur that changes my sector preference?
3. Did any macro data change the interest rate / inflation outlook?
4. Did any geopolitical event change the risk landscape?
5. Did any earnings report change my thesis on a specific stock?

### If YES to Any Question
- Update the relevant brain files in `/Brain/`
- Update the market regime detector if regime changed
- Update the sector rotation model if flows shifted
- Note the thesis change in the trading journal

### If NO to All Questions
- Good. Thesis intact. Stay the course.
- Log: "Thesis unchanged. [Brief reason]."

---

## Step 10: Post the Day's Recap (If During Posting Hours)
**Time: 5 minutes**

If it is between 1:00 PM and 8:00 PM PT, draft a market recap for social media.

### Recap Format
```
ORACLE Market Recap — [DATE]

Markets: SPY [+/-X%] | QQQ [+/-X%] | BTC [+/-X%]
Sector Leaders: [Top 2 sectors]
Sector Laggards: [Bottom 2 sectors]

Key Move: [The one thing that mattered most today]

ORACLE's Take: [1-2 sentences on what this means going forward]

Tomorrow: [Key catalyst to watch]
```

### Rules
- Only post if you have something worth saying. No filler content.
- Be honest about what happened. Do not spin losses as wins.
- Include one actionable insight, not just a recap.

---

## Step 11: Study One New Thing
**Time: 10 minutes**

Every single day, learn something new. This compounds over months and years.

### Study Topics (rotate through)
- **Monday:** Options strategy (read one setup from options-mastery-path.md)
- **Tuesday:** Technical analysis (study one chart pattern on a stock from your universe)
- **Wednesday:** Macro economics (read one Fed speech, economic report, or macro analysis)
- **Thursday:** Crypto on-chain analysis (study one on-chain metric from Glassnode)
- **Friday:** Trade review (deep-dive one trade from this week — what would you do differently?)
- **Weekend:** Read one chapter of a trading book or watch one educational video

### Rules
- 10 minutes minimum. Set a timer.
- Take ONE note in the lessons-learned.md file.
- The note should be actionable: "I learned X, which means I should Y when Z happens."

---

## Step 12: Update Brain Files
**Time: 5 minutes**

Check if any of these files need updates based on today's activity:

| File | Update If... |
|------|-------------|
| `lessons-learned.md` | You learned something new or made a mistake worth documenting |
| `tiered-watchlist.md` | Any watchlist changes needed (already done in Step 6, but double-check) |
| Tracker files in `/Market/` | Any material news, price changes, or thesis updates on covered names |
| `risk-dashboard.md` | Risk levels changed (position sizes, correlation, drawdown) |
| `trading-journal.md` | Trades logged (already done in Step 1, but verify completeness) |
| `sector-rotation-model.md` | Sector flows changed today |

---

## Step 13: Commit and Push Vault
**Time: 1 minute**

All changes made during the session need to be saved and synced.

### Checklist
- [ ] All files saved
- [ ] Vault synced (if using Obsidian Sync or Git)
- [ ] Tomorrow's workspace is clean and ready
- [ ] Browser tabs closed (start fresh tomorrow)
- [ ] Alerts set (Step 8 confirmed)

---

## End-of-Day Routine Completion Log

```
### EOD Routine — [DATE] — Completed at [TIME] PT

| Step | Done | Notes |
|------|------|-------|
| 1. Log trades | [ ] | |
| 2. Grade trades | [ ] | |
| 3. Update P&L | [ ] | |
| 4. Stress test | [ ] | |
| 5. AH earnings | [ ] | |
| 6. HOT watchlist | [ ] | |
| 7. Tomorrow catalysts | [ ] | |
| 8. Price alerts | [ ] | |
| 9. Thesis check | [ ] | |
| 10. Recap post | [ ] | |
| 11. Study | [ ] | |
| 12. Brain files | [ ] | |
| 13. Commit/push | [ ] | |

**Time Spent:** [minutes]
**Overall Day Grade:** [A/B/C/F]
**Tomorrow's Focus:** [One sentence]
```

---

## Non-Negotiable Rules

1. **Do this routine EVERY day.** Not most days. EVERY day. Consistency is the edge.
2. **Do it IMMEDIATELY after close.** Not "later tonight." The further you get from the session, the less you remember and the less you care.
3. **Be honest in the journal.** Nobody else is reading it. If you revenge-traded, write it down. If you FOMO'd, write it down. Honesty is how you improve.
4. **The routine takes 45-60 minutes.** If you cannot commit 45 minutes to improving, you should not be trading.
5. **If you skip this routine 3 days in a row, STOP TRADING.** You have lost discipline. Take a break, reset, and come back when you can commit to the process.
