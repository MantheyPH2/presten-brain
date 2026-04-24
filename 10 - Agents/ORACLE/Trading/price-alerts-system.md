---
type: trading-tool
agent: ORACLE
title: Price Alert & Notification System
created: 2026-04-23
---

# Price Alert & Notification System

**ORACLE runs every 15 minutes. But prices can move 5% in 5 minutes. The gap between sessions needs to be covered by alerts.**

---

## How ORACLE Monitors Between Sessions

Since ORACLE runs as a scheduled process (not a persistent WebSocket connection), he needs to set up monitoring that works BETWEEN his sessions:

### Method 1: WebSearch Price Checks (Every Session)
Every 15-minute session, ORACLE should:
1. Check current price of EVERY open position
2. Compare to stops and targets from watchlist.md
3. If any stop or target is within 2% of being hit → FLAG IT
4. If any stop was breached between sessions → LOG IT and update position

### Method 2: Watchlist Alert Levels
Maintain in `memory/watchlist.md` for every open position:

```
ALERT LEVELS:
BTC: STOP $75,500 | TARGET $82,000 | WARNING at $76,000 (within 2%)
AMZN: STOP $240 | TARGET $280 | WARNING at $245
META: STOP $640 | TARGET $700 | WARNING at $650
```

Every session: WebSearch "[ticker] price" → compare to alert levels → if WARNING zone or BREACHED → take action immediately.

### Method 3: Key Market-Wide Alerts
Not just positions — watch these market levels:

| Alert | Trigger | Action |
|-------|---------|--------|
| VIX crosses 25 | Fear rising | Check all positions, consider hedges |
| VIX crosses 30 | Panic building | Crash playbook Phase 1 |
| VIX crosses 40 | Full panic | Mean reversion prep — THIS IS THE MOMENT |
| SPY drops 2% intraday | Selloff | Check all longs, tighten stops |
| SPY drops 5% | Crash | Full crash playbook |
| BTC drops 10% in 24h | Crypto selloff | Check all crypto, assess contagion |
| BTC funding rate flips positive after extended negative | Squeeze ending | Reassess BTC position |

### Method 4: Earnings Auto-Alert
Maintain an earnings calendar and every session check:
- Is any stock in my watchlist reporting TODAY or TOMORROW?
- If yes → run earnings whisper checklist
- If reporting today → DO NOT enter new positions on that name until after the print

---

## Event Calendar (Check Every Session)

```
UPCOMING EVENTS — CHECK IF DATES CHANGED:
Apr 29: MSFT, AMZN, META, GOOGL earnings
Apr 30: PCE inflation 5:30 AM PT
May 1: ISM Manufacturing PMI
May 4: SUI CME futures
May 20: NVDA earnings
```

Every session: "Is today or tomorrow an event day?" If yes → position accordingly.

---

## Notification to Presten

When ORACLE detects something urgent, post to the queue:
`10 - Agents/ORACLE/Queue/pending-YYYY-MM-DD-{urgency}.md`

Categories:
- **URGENT:** Stop hit, position needs exit, crash starting, major news
- **ALERT:** Warning zone reached, indicator changed, event tomorrow
- **UPDATE:** Regular P&L update, thesis change, new setup found

Also tweet urgent market moves to @mantheyXx — Presten follows the account and sees it immediately.

---

## What ORACLE Checks Every Single Session (15-min cadence)

```
SESSION QUICK CHECK (2 minutes if nothing happened):
□ Date/time? (what market phase am I in)
□ BTC price? (crypto never sleeps)
□ SPY/QQQ futures or last close? (equity direction)
□ VIX level? (fear gauge)
□ Any position within 2% of stop or target?
□ Any earnings/events TODAY?
□ Any breaking news? (quick WebSearch)
□ All clear → done. See you in 15 minutes.
□ Something moved → full session, investigate and act.
```
