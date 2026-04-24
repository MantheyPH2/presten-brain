---
type: trading-tool
agent: ORACLE
title: Morning & Evening Brief Protocol
created: 2026-04-21
last_updated: "{{date}}"
---

# Morning & Evening Brief Protocol

**ORACLE delivers a brief every morning and every evening. Presten should never have to open Obsidian to know what matters. The tweet hits his phone. The vault has the full detail if he wants it. The queue catches anything urgent.**

---

## Morning Brief — Delivered by 5:30 AM PT Every Trading Day

### Step 1: Gather Data (5:00-5:15 AM PT)

Run these searches to get the raw data for the brief:

```
WebSearch "S&P 500 futures pre-market"
WebSearch "Nasdaq futures pre-market"
WebSearch "VIX level"
WebSearch "BTC price"
WebSearch "10 year treasury yield"
WebSearch "oil price WTI"
WebSearch "pre-market movers today"
WebSearch "economic calendar today"
WebSearch "stock market news overnight"
```

For each open position:
```
WebSearch "[ticker] pre-market price" (for stocks)
WebSearch "[ticker] price" (for crypto — trades 24/7)
```

### Step 2: Assess Overnight Moves (5:15-5:20 AM PT)

Fill in this assessment:

```
OVERNIGHT ASSESSMENT: {{date}}

CRYPTO (traded overnight):
  BTC: $_______ (____% from yesterday close)
  SUI: $_______ (____% from yesterday close)  
  RENDER: $_______ (____% from yesterday close)
  AAVE: $_______ (____% from yesterday close)
  Any flash crashes? ____
  Any significant moves (>3%)? ____

FUTURES (pre-market):
  SPY futures: _______ (____%)
  QQQ futures: _______ (____%)
  Direction bias: BULLISH / BEARISH / FLAT

MACRO:
  VIX: _______
  10Y yield: _______%
  Oil (WTI): $_______
  DXY: _______
  Any overnight news that moves markets: ____________________

POSITIONS:
  Any stop threatened? YES / NO → which one?
  Any target about to be hit? YES / NO → which one?
  Overall position status: ALL GREEN / WARNING / CRITICAL
```

### Step 3: Write the Tweet (5:20-5:25 AM PT)

This is what Presten sees on his phone. It must be concise, informative, and actionable. One tweet. Under 280 characters if possible, thread if needed.

**Template:**
```
Good morning. Here's what matters today:

BTC: $[price] ([change]%)
SPY futures: [change]%
VIX: [level]

[Key event or catalyst if any]

[One-line portfolio status]

[One-line watchlist highlight or risk note]
```

**Examples of good morning tweets:**

When everything is calm:
```
Good morning. Markets quiet overnight.

BTC: $78.2K (+0.8%)
SPY futures: +0.2%
VIX: 17.5

No major data today. Watching AMZN for earnings prep — reports Thursday.

All positions green. Stops intact.
```

When something is happening:
```
MORNING ALERT.

BTC flash-crashed to $72K at 3 AM, recovered to $74.8K. 
SPY futures: -0.6%
VIX: 22.3

BTC stop at $75.5K was BREACHED. Monitoring for re-entry.
PCE data at 5:30 AM — hot print expected.

Reducing risk across the board.
```

When there is a big event:
```
FOMC day. Here's the playbook:

BTC: $80.1K (+1.2%)
SPY futures: flat (waiting)
VIX: 19.2

Decision at 11 AM PT. Market expects hold.
If hawkish surprise → sell tech, add hedge.
If dovish → risk-on rally, add to winners.

No trades until after the press conference settles.
```

### Step 4: Write Full Brief to Vault (5:25-5:30 AM PT)

Save to: `ORACLE/Briefings/YYYY-MM-DD-morning.md`

**Full Brief Template:**

```markdown
---
type: briefing
period: morning
date: {{date}}
market_bias: BULLISH / BEARISH / NEUTRAL
risk_level: LOW / MEDIUM / HIGH / CRITICAL
---

# Morning Brief — {{date}}

## Overnight Summary
[2-3 sentences on what happened overnight. Crypto moves, Asia/Europe session, any breaking news.]

## Market Snapshot
| Indicator | Level | Change | Signal |
|-----------|-------|--------|--------|
| SPY futures | | | |
| QQQ futures | | | |
| BTC | | | |
| VIX | | | |
| 10Y yield | | | |
| DXY | | | |
| Oil (WTI) | | | |

## Position Status
| Ticker | Current | vs Entry | Stop Status | Notes |
|--------|---------|----------|-------------|-------|
| META | | | | |
| AAVE | | | | |
| BTC | | | | |
| RENDER | | | | |
| AMZN | | | | |
| SUI | | | | |
| NVDA | | | | |

## Today's Events
| Time (PT) | Event | Expected Impact | My Plan |
|-----------|-------|----------------|---------|
| | | | |

## Today's Plan
### Watching For:
- [Setup 1]
- [Setup 2]

### Will Execute If:
- [Condition → Action]
- [Condition → Action]

### Risk Notes:
- [Any elevated risks today]

## Crash Scorecard
[Current crash probability assessment from market-regime-detector]
Score: ___/10
```

### Step 5: Check Queue for Urgency
If anything from the assessment warrants Presten's immediate attention:
- Stop breached overnight → Write to `ORACLE/Queue/` with `priority: urgent`
- Major news that changes a thesis → Write to `ORACLE/Queue/` with `priority: high`
- Event today that requires a decision → Write to `ORACLE/Queue/` with `priority: high`

---

## Evening Brief — Delivered by 5:00 PM PT Every Trading Day

### Step 1: Gather End-of-Day Data (4:00-4:15 PM PT)

```
WebSearch "S&P 500 close today"
WebSearch "Nasdaq close today"
WebSearch "VIX close today"
WebSearch "market recap today"
WebSearch "after hours movers"
```

For each position:
```
WebSearch "[ticker] closing price today"
```

### Step 2: Calculate Daily Performance
Run the [[auto-pl-calculator]] with closing prices. Record:
- Each position's daily change
- Total portfolio daily change
- Any stops hit during the day
- Any targets hit during the day

### Step 3: Write the Evening Tweet

**Template:**
```
Market close: [date]

SPY: [close] ([change]%)
QQQ: [close] ([change]%)
VIX: [close]

Portfolio: [change]% today | [total P&L]% all-time

[Biggest winner/loser today]
[After-hours earnings reaction if any]

Tomorrow: [one-line preview]
```

### Step 4: Write Full Evening Brief to Vault

Save to: `ORACLE/Briefings/YYYY-MM-DD-evening.md`

```markdown
---
type: briefing
period: evening
date: {{date}}
market_result: UP / DOWN / FLAT
portfolio_daily: +/-____%
---

# Evening Brief — {{date}}

## Market Summary
[What happened today in 2-3 sentences. What drove the market. What was unexpected.]

## Daily Scorecard
| Indicator | Open | Close | Change | Signal |
|-----------|------|-------|--------|--------|
| SPY | | | | |
| QQQ | | | | |
| BTC | | | | |
| VIX | | | | |

## Portfolio Performance
| Ticker | Open | Close | Daily Change | Total P&L | Notes |
|--------|------|-------|-------------|----------|-------|
| META | | | | | |
| AAVE | | | | | |
| BTC | | | | | |
| RENDER | | | | | |
| AMZN | | | | | |
| SUI | | | | | |
| NVDA | | | | | |
| TOTAL | | | | | |

## Trades Executed Today
| Time | Ticker | Action | Price | Reason |
|------|--------|--------|-------|--------|
| | | | | |

## After-Hours Activity
[Any earnings results, after-hours movers, breaking news after close]

## Tomorrow's Setup
### Key Events:
- [Event 1]
- [Event 2]

### Positions to Watch:
- [Position 1: why]
- [Position 2: why]

### Pre-Market Actions:
- [What ORACLE will do in the morning session]
```

---

## Presten's Daily Routine — 2 Minutes Total

This is the entire time commitment Presten needs to spend on markets each day, because ORACLE handles everything else.

### Morning (1 minute)
1. Wake up. Check phone.
2. See @mantheyXx morning tweet on timeline. Read it.
3. **All clear?** Go about your day. ORACLE is on it.
4. **Something flagged?** Open Obsidian. Check `ORACLE/Queue/` for details. Make any decisions ORACLE needs.

### Evening (1 minute)
1. Check phone before bed.
2. See @mantheyXx evening tweet. Portfolio up or down? By how much?
3. **All clear?** Sleep well. ORACLE will monitor crypto overnight.
4. **Action needed?** Open Obsidian. Review the evening brief. Make decisions.

### That's it.
No chart-staring. No news-scrolling. No CNBC. No Reddit trading threads. ORACLE does all of that. Presten makes high-level decisions when ORACLE surfaces them. Everything else is automated.

---

## Weekend Brief — Delivered Saturday Morning

### Saturday Morning (weekly review)
One comprehensive tweet + full vault brief covering:
- This week's portfolio performance (P&L, win rate)
- Best and worst positions
- Key events that happened
- Next week's calendar preview
- Any thesis changes or position adjustments needed
- One-line conviction check: "Still bullish because ___" or "Turning cautious because ___"

Save to: `ORACLE/Briefings/YYYY-MM-DD-weekly.md`

---

## Brief Quality Standards

Every brief must meet these standards or it is not published:

1. **Data is FRESH.** Every number comes from a WebSearch in this session. No stale data. No guessing.
2. **Actionable.** Presten should know what to do (or that he needs to do nothing) after reading the tweet.
3. **Honest.** If the portfolio is losing money, say so. If a thesis is failing, flag it. No hopium.
4. **Concise.** The tweet is under 280 characters when possible. The vault brief is under 1 page. Brevity is respect for Presten's time.
5. **Consistent.** Same format every day. Presten should never have to hunt for information — it is always in the same place.

---

## Briefing Archive Structure

```
ORACLE/Briefings/
├── 2026-04-21-morning.md
├── 2026-04-21-evening.md
├── 2026-04-22-morning.md
├── 2026-04-22-evening.md
├── 2026-04-26-weekly.md
└── ...
```

This archive becomes valuable over time. ORACLE can reference past briefs to see how the market evolved, whether predictions were correct, and how the portfolio responded to events. Pattern recognition improves with data.
