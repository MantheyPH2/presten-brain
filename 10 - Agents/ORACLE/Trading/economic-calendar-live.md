---
type: trading-tool
agent: ORACLE
title: Economic Calendar (Live)
created: 2026-04-21
last_updated: "{{date}}"
---

# Economic Calendar — LIVE

**ORACLE updates this every Monday and after every major release. This is not a static reference. It is a live document with actual results, market reactions, and ORACLE's actions logged.**

---

## This Week's Events

**Week of: ____________ (ORACLE fills this every Monday morning)**

| Date | Time (PT) | Event | Consensus | Previous | Actual | Beat/Miss | Market Reaction | ORACLE Action Taken |
|------|-----------|-------|-----------|----------|--------|-----------|----------------|-------------------|
| | | | | | | | | |
| | | | | | | | | |
| | | | | | | | | |
| | | | | | | | | |
| | | | | | | | | |
| | | | | | | | | |
| | | | | | | | | |

### How to Populate Every Monday
1. `WebSearch "economic calendar this week"` — get all macro releases
2. `WebSearch "earnings calendar this week"` — get all major earnings
3. Fill in Date, Time (convert to PT), Event name, Consensus estimate, Previous reading
4. Leave Actual, Beat/Miss, Market Reaction, and ORACLE Action blank until the event happens
5. After each event releases: `WebSearch "[event name] result today"` → fill in remaining columns

---

## This Week's Earnings

| Date | Before/After | Company | Sector | Relevance to Portfolio | Notes |
|------|-------------|---------|--------|----------------------|-------|
| | | | | | |
| | | | | | |
| | | | | | |

**Relevance scoring:**
- **DIRECT** — We hold this name (e.g., AMZN, META, NVDA)
- **HIGH** — Same sector/industry as a position (e.g., GOOGL affects META)
- **MEDIUM** — Bellwether for the market (e.g., AAPL, MSFT)
- **LOW** — Interesting but no direct portfolio impact

---

## Major Recurring Events — Know These by Heart

These events move markets every single time. ORACLE should have dates memorized and preparation started days in advance.

### FOMC Meetings (8 per year)
The single most important recurring event. Rate decision + statement + press conference (if scheduled). Dates are published a year in advance.

| Meeting | Date | Rate Decision | Statement Tone | Market Reaction |
|---------|------|--------------|----------------|-----------------|
| Jan | | | | |
| Mar | | | | |
| May | | | | |
| Jun | | | | |
| Jul | | | | |
| Sep | | | | |
| Nov | | | | |
| Dec | | | | |

**FOMC Protocol:**
- 2 weeks before: Start monitoring Fed funds futures for rate expectations
- 1 week before: Reduce position sizes if uncertain about direction
- Day before: Set QQQ hedge if not already in place
- Decision day: Do NOT trade in the 30 minutes before announcement (2:00 PM ET / 11:00 AM PT). Wait for the press conference reaction to settle (usually 30-60 min after).
- Day after: Assess. Did the market accept the decision? Follow-through or reversal?

### CPI — Consumer Price Index
**When:** Monthly, approximately the 12th, 5:30 AM PT
**What it measures:** Inflation from the consumer side
**Why it matters:** Hot CPI = Fed stays hawkish = growth stocks sell off. Cool CPI = rate cuts closer = risk-on rally.
**Key numbers:** Core CPI (ex food and energy) matters more than headline.

### PCE — Personal Consumption Expenditures
**When:** Monthly, approximately the last Friday of the month, 5:30 AM PT
**What it measures:** The Fed's PREFERRED inflation gauge
**Why it matters:** This is what the Fed actually targets. More important than CPI for policy decisions.
**Key numbers:** Core PCE year-over-year. Fed target is 2.0%.

### NFP — Non-Farm Payrolls (Jobs Report)
**When:** First Friday of each month, 5:30 AM PT
**What it measures:** How many jobs the economy added/lost
**Why it matters:** Strong jobs = economy healthy but Fed may stay tight. Weak jobs = recession fears but rate cuts more likely. Goldilocks (moderate growth) is the best outcome.
**Key numbers:** Headline number, unemployment rate, average hourly earnings (wage inflation).

### GDP — Gross Domestic Product
**When:** Quarterly. Three readings: advance (most important), preliminary, final
**What it measures:** Total economic output
**Why it matters:** Negative GDP = recession narrative. Strong GDP = economy resilient.

### ISM Manufacturing & Services
**When:** Manufacturing: first business day of month. Services: third business day of month.
**What it measures:** Purchasing managers' outlook. Above 50 = expansion. Below 50 = contraction.
**Why it matters:** Leading indicator. If businesses are pulling back on orders, the economy is slowing.

### Retail Sales
**When:** Approximately the 15th of each month
**What it measures:** Consumer spending
**Why it matters:** Consumer spending is ~70% of US GDP. Weak retail = economy weakening.

### FOMC Minutes
**When:** 3 weeks after each FOMC meeting
**What it measures:** Detailed discussion from the meeting
**Why it matters:** Reveals dissent, debate, and nuance that the statement didn't capture. Can move markets if the tone differs from what was expected.

### Jackson Hole Symposium
**When:** Late August, annually
**What it matters:** Fed Chair's speech here has historically signaled major policy shifts. "Whatever it takes" moments happen here.

### Quarterly Options Expiration (Quad Witching)
**When:** Third Friday of March, June, September, December
**What it matters:** Massive volume and volatility as options, futures, and index options all expire. Position for increased volatility the week before.

---

## Pre-Event Protocol

Run this the DAY BEFORE any major event (CPI, PCE, NFP, FOMC, GDP, or direct-holding earnings):

### 1. Consensus Check
```
WebSearch "[event] consensus estimate"
WebSearch "[event] whisper number"
```
- What does the market expect?
- Is there a whisper number that differs from consensus?
- What are options implying for the move size? (`WebSearch "[ticker] implied move [event]"`)

### 2. Scenario Planning
For EVERY major event, have TWO plans ready before the data drops:

**Scenario A: Beat/Hot (above consensus)**
- Which positions benefit?
- Which positions get hurt?
- What trades do I make?
- At what price do I act?

**Scenario B: Miss/Cool (below consensus)**
- Which positions benefit?
- Which positions get hurt?
- What trades do I make?
- At what price do I act?

**Scenario C: In-line (matches consensus)**
- Usually a non-event. Hold positions. Watch for "sell the news" reaction.

### 3. Position Sizing Adjustment
- Uncertain about direction? Reduce position sizes by 25-50% the day before.
- High conviction on the outcome? Maintain full size, set stops tight.
- Never INCREASE size into an event unless conviction is 5/5 AND the risk/reward is clearly defined.

### 4. Hedge Check
- Is the QQQ hedge in place? If not, consider entering one before CPI/PCE/FOMC.
- Are stops on all positions current and appropriate?
- What is max portfolio loss if the event goes against every position?

### 5. Log the Plan
Write the pre-event plan to the session log so post-event ORACLE can execute it without re-analyzing:
```
EVENT: [name]
DATE: [date]
CONSENSUS: [number]
IF BEATS: [action]
IF MISSES: [action]
IF INLINE: [action]
HEDGE STATUS: [in place / not needed / entering now]
```

---

## Post-Event Protocol

Run this WITHIN 15 MINUTES of any major data release:

### 1. Get the Number
```
WebSearch "[event] actual result today"
```

### 2. Beat or Miss?
- Compare actual to consensus AND to whisper number
- A "beat" vs consensus but "miss" vs whisper can still sell off

### 3. Market Reaction
```
WebSearch "SPY price" 
WebSearch "QQQ price"
WebSearch "VIX level"
WebSearch "10 year treasury yield"
```
- Which direction is the market moving?
- Is the reaction logical? (Hot CPI should = sell tech. If tech is rallying on hot CPI, something else is driving it.)
- How big is the move vs implied move?

### 4. Execute Pre-Planned Response
- Pull up the pre-event plan (logged above)
- Execute Scenario A, B, or C as appropriate
- Do NOT deviate from the plan unless the market reaction is completely unexpected

### 5. Tweet the Reaction
Post a take within 30 minutes of the release:
```
[EVENT] came in at [ACTUAL] vs [CONSENSUS] expected.

[Hot/Cool/Inline]. Market reaction: [direction].

My read: [1-2 sentence analysis].

Positions: [what I'm doing].
```

### 6. Update This Calendar
Fill in the Actual, Beat/Miss, Market Reaction, and ORACLE Action columns in the weekly table above.

---

## Historical Results Tracker

After each month, archive the weekly tables here for pattern recognition:

### April 2026
| Date | Event | Consensus | Actual | Beat/Miss | Market Reaction |
|------|-------|-----------|--------|-----------|-----------------|
| | | | | | |

### May 2026
| Date | Event | Consensus | Actual | Beat/Miss | Market Reaction |
|------|-------|-----------|--------|-----------|-----------------|
| | | | | | |

Over time, this archive reveals which events actually move markets for THIS portfolio and which are noise. Use that data to refine the pre-event protocol.
