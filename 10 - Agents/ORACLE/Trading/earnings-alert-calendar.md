---
type: trading-tool
agent: ORACLE
title: Earnings Alert Calendar
created: 2026-04-21
last_updated: "{{date}}"
---

# Earnings Alert Calendar

**ORACLE tracks every earnings date that matters to the portfolio. Preparation starts 7 days out, not 7 hours. This calendar auto-triggers preparation steps at defined intervals before each report.**

---

## Current Earnings Season Tracker

**Season: Q1 2026 (reporting April-May 2026)**

| Date | Before/After | Company | Sector | Consensus EPS | Whisper EPS | Revenue Est | Implied Move | IV Rank | My Play | Result | P&L |
|------|-------------|---------|--------|---------------|-------------|-------------|-------------|---------|---------|--------|-----|
| | | | | | | | | | | | |
| | | | | | | | | | | | |
| | | | | | | | | | | | |
| | | | | | | | | | | | |
| | | | | | | | | | | | |

**How to populate:**
```
WebSearch "[company] earnings date Q1 2026"
WebSearch "[company] earnings consensus EPS"
WebSearch "[company] earnings whisper"
WebSearch "[company] implied move earnings"
WebSearch "[company] IV rank"
```

---

## Auto-Trigger Rules

These triggers fire automatically based on the number of days until a tracked company reports. ORACLE checks the calendar every session and runs the appropriate trigger.

### T-7 Days: Early Prep
**Trigger:** 7 calendar days before earnings date
**Actions:**
1. Add company to HOT watchlist if not already there
2. Start monitoring IV rank daily — record it in the table above
3. `WebSearch "[company] earnings preview"` — read analyst expectations
4. Note any unusual options activity (`WebSearch "[company] unusual options activity"`)
5. Check: do we hold this name directly? If yes, decision point coming.

### T-3 Days: Earnings Whisper Checklist
**Trigger:** 3 business days before earnings
**Actions:**
1. Run the full earnings whisper checklist (see [[earnings-whisper-system]])
2. `WebSearch "[company] earnings whisper number"` — get the real expectation
3. Compare whisper to consensus. If whisper is significantly above consensus, the stock needs to beat the whisper, not just consensus.
4. Check the company's earnings history: do they typically beat or miss? By how much?
5. **DECIDE: Trade this earnings or skip?**
   - Trading it → define the exact play (debit spread, straddle, directional bet, etc.)
   - Skipping it → document why. No FOMO allowed.

### T-1 Day: Final Checklist
**Trigger:** 1 business day before earnings
**Actions:**
1. **Final IV rank check.** If IV rank is above 80, selling premium is statistically favorable. Below 30, buying premium has edge.
2. **Enter position if trading it.** Do NOT enter on the day of — IV crush and emotion make day-of entries worse.
3. **Set stops and targets.** Know exactly where you exit on both sides.
4. **Log the trade plan:**
   ```
   EARNINGS TRADE PLAN
   Company: ____
   Date: ____
   Direction: Long / Short / Neutral
   Vehicle: [shares / calls / puts / spread / straddle]
   Entry: $____
   Stop: $____
   Target: $____
   Max loss: $____
   If beats: [action]
   If misses: [action]
   If in-line: [action]
   ```
5. **Portfolio check:** With this trade, is total portfolio heat still under control?

### T-0 Day (Before Open): Pre-Market Check
**Trigger:** Morning of earnings day, if company reports before market open
**Actions:**
1. `WebSearch "[company] earnings results"` — get the numbers
2. `WebSearch "[company] pre-market price"` — see the gap
3. Compare actual EPS and revenue to consensus AND whisper
4. Is the gap up/down bigger or smaller than the implied move?
5. **If we are already in a position:** Does the result confirm or invalidate our thesis?
6. **If gap is larger than implied move:** Be cautious — mean reversion is likely intraday

### T-0 Day (After Close): Immediate Reaction
**Trigger:** Evening of earnings day, if company reports after market close
**Actions:**
1. `WebSearch "[company] earnings results"` — get the numbers immediately
2. `WebSearch "[company] after hours price"` — see the reaction
3. Read the earnings call highlights (`WebSearch "[company] earnings call highlights"`)
4. Key metrics beyond EPS:
   - Revenue growth rate
   - Guidance (raised / maintained / lowered — THIS matters more than the beat)
   - Margins (expanding or compressing?)
   - Any one-time items inflating/deflating the number?
5. **Alert Presten if the move is significant** (tweet + queue item)
6. Log the result in the tracker table above

### T+1 Day: Post-Earnings Assessment
**Trigger:** 1 business day after earnings
**Actions:**
1. What did the stock do on the first full trading day after earnings?
2. **Gap and go** (continued in the direction of the gap) → trend confirmation, hold/add
3. **Gap and fade** (reversed the gap) → market disagrees with initial reaction, reassess
4. **Gap fill** (returned to pre-earnings price) → earnings were a non-event, remove from watch
5. Update the Result and P&L columns in the tracker
6. Log in [[trading-journal]] and [[post-trade-review]] if we traded it

---

## Mega-Cap Earnings Dates — Always Track These

These companies move the entire market. Even if we do not hold them, their results affect our positions. ORACLE maintains this list rolling 3 months forward.

### How to Maintain
Every 4 weeks: `WebSearch "mega cap earnings calendar next 3 months"` and update this table.

### Rolling Mega-Cap Calendar

| Company | Ticker | Expected Date | Holds? | Sector Impact | Last Updated |
|---------|--------|--------------|--------|---------------|-------------|
| Apple | AAPL | | No | Tech/QQQ weight | |
| Microsoft | MSFT | | No | Tech/QQQ/Cloud | |
| Amazon | AMZN | | YES | Tech/Consumer/Cloud | |
| Alphabet | GOOGL | | No | Tech/Ads/AI | |
| Meta | META | | YES | Tech/Ads/AI | |
| NVIDIA | NVDA | | YES | AI/Semis | |
| Tesla | TSLA | | No | EV/Momentum | |
| Broadcom | AVGO | | No | Semis/AI | |
| AMD | AMD | | No | Semis/AI | |
| Netflix | NFLX | | No | Tech/Consumer | |
| Coinbase | COIN | | No | Crypto proxy | |
| MicroStrategy | MSTR | | No | BTC proxy | |

### Sector Bellwethers
When these report, pay attention even if we don't hold them — they set the tone for our positions:

| Bellwether | Reports For | Our Exposed Position |
|-----------|------------|---------------------|
| AAPL | Consumer tech health | META, AMZN |
| TSM | Semiconductor supply | NVDA |
| JPM/GS | Financial health, deal flow | Market sentiment |
| WMT/TGT | Consumer spending | AMZN |
| GOOGL | Digital ad spending | META |
| AMD | AI demand beyond NVDA | NVDA, RENDER |

---

## Earnings Season Calendar

Major earnings windows (mark these on the calendar and expect elevated volatility):

| Season | Approximate Window | Peak Week |
|--------|-------------------|-----------|
| Q4 reports | Mid-January to mid-February | Last week of January |
| Q1 reports | Mid-April to mid-May | Last week of April |
| Q2 reports | Mid-July to mid-August | Last week of July |
| Q3 reports | Mid-October to mid-November | Last week of October |

**During peak earnings weeks:**
- Expect daily 1-2% index moves
- IV is elevated across the board — selling premium can be profitable
- Correlation between stocks drops (idiosyncratic moves dominate)
- Reduce position sizes if uncertain, increase if conviction is high on specific names

---

## Historical Earnings Results Archive

After each season, archive results here for pattern analysis:

### Q1 2026 Season (April-May 2026)
| Company | Date | EPS Est | EPS Actual | Revenue Est | Revenue Actual | Guidance | Day-After Move | My Trade | My P&L |
|---------|------|---------|------------|-------------|----------------|----------|---------------|----------|--------|
| | | | | | | | | | |

Over time, this archive answers:
- Which companies consistently beat whisper numbers?
- Which companies' guidance matters more than the beat?
- What is the average implied move vs actual move? (Systematically selling overpriced straddles becomes possible with this data.)
- What is my earnings trading win rate? Am I actually profitable on earnings plays?
