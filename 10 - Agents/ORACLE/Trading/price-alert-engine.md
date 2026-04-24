---
type: trading-tool
agent: ORACLE
title: Price Alert Engine
created: 2026-04-21
last_run: "{{date}} {{time}}"
---

# Price Alert Engine

**This is not a notification app. This is a PROCESS. Every 15-minute session, ORACLE runs this engine to catch every price move that matters. No exceptions. No shortcuts. If a stop gets breached and ORACLE missed it, this process failed.**

---

## Alert Levels Table

ORACLE maintains this table and checks it EVERY session. Every open position and every HOT watchlist item gets a row. If it is not in this table, it is not being monitored.

### Open Positions (update prices every session)

| Ticker | Direction | Entry | Current | Stop | Target | Warning Zone | Status |
|--------|-----------|-------|---------|------|--------|-------------|--------|
| META | Long | $565 | | $640 | $700 | $652-$640 | |
| AAVE | Long | $88 | | $91 | $105 | $92.82-$91 | |
| BTC | Long | $74,000 | | $75,500 | $85,000 | $77,010-$75,500 | |
| RENDER | Long | $1.72 | | $1.55 | $2.50 | $1.581-$1.55 | |
| AMZN | Long | $248 | | $232 | $280 | $236.64-$232 | |
| SUI | Long | $0.93 | | $0.88 | $1.15 | $0.898-$0.88 | |
| NVDA | Long | $198 | | $185 | $240 | $188.70-$185 | |

### HOT Watchlist Items (update when names change)

| Ticker | Setup | Key Level | Warning Zone | Status |
|--------|-------|-----------|-------------|--------|
| | | | | |
| | | | | |

**Warning Zone = within 2% of stop or target. Calculated as: stop x 1.02 (for longs) or target x 0.98 (for longs).**

---

## Market-Wide Alert Levels

These are non-negotiable triggers. When any of these fires, ORACLE acts immediately regardless of what else is happening in the session.

| Level | Trigger | What ORACLE Does |
|-------|---------|-----------------|
| VIX 20 | Caution | Tighten all stops 5% closer to current price. Log the tightening. |
| VIX 25 | Warning | Reduce all long exposure by 30%. Exit weakest-conviction positions first. Tweet the risk-off shift. |
| VIX 30 | Danger | Activate crash playbook Phase 1. No new longs. Hedge everything. Full portfolio review. |
| VIX 40+ declining | OPPORTUNITY | VIX mean-reversion play. Assess aggressively buying dips on highest-conviction names. Size up. |
| SPY -2% intraday | Selloff | Check every open position. Any stop threatened? Tighten trailing stops. |
| SPY -5% intraday | Crash | Full crash playbook. Exit all positions without conviction 4+/5. Keep only hedges and highest-conviction longs. |
| BTC -5% in 1 hour | Flash crash | Check all crypto positions (BTC, SUI, RENDER, AAVE). Assess if this is liquidation cascade or fundamental. |
| BTC -10% in 24 hours | Major selloff | Assess contagion risk to equities. Check if QQQ/SPY also falling. Reduce crypto if correlated selling. |
| Any position -50% from entry | Emergency | EXIT IMMEDIATELY. No questions. No "let me think about it." This is a catastrophic failure of risk management that should never happen with proper stops. |
| 10Y yield +20bps in 1 day | Rate shock | Check all growth/tech positions. These sell off hardest on rate spikes. Consider adding hedge. |
| DXY +2% in 1 week | Dollar surge | Risk-off signal. Check crypto and international exposure. |

---

## How ORACLE Alerts Presten

ORACLE cannot send push notifications. But Presten needs to know when something urgent happens. Here is the multi-channel system:

### Channel 1: Tweet It
Presten follows @mantheyXx. This is the fastest way to get his attention because he checks his phone.
- **Urgent market move** (stop breached, VIX spike, flash crash) → Tweet immediately with the facts and what ORACLE is doing about it.
- Format: `ALERT: [TICKER] hit stop at $[PRICE]. Exiting position. Details in vault.`

### Channel 2: Queue Item
Write to `ORACLE/Queue/` with appropriate priority.
- `priority: urgent` — stop breached, crash conditions, position emergency
- `priority: high` — warning zone entered, market-wide alert triggered
- `priority: normal` — informational, position approaching level

Queue file format:
```
---
priority: urgent
created: {{date}} {{time}}
type: price-alert
---

# [TICKER] Stop Breached

Current price: $____
Stop level: $____
Entry price: $____
P&L at exit: ____%

Action taken: [what ORACLE did]
Action needed from Presten: [what Presten needs to do, if anything]
```

### Channel 3: Update watchlist.md
Flag the position status in the tiered watchlist and risk dashboard.
- Stop breached → mark as STOPPED OUT, remove from open positions
- Warning zone → mark as WARNING, highlight the row
- Target hit → mark as TARGET HIT, log the win

### Channel 4: All Three Simultaneously
For truly urgent events (crash conditions, emergency exits, position down 50%), fire ALL three channels at once:
1. Tweet the alert
2. Write urgent queue item
3. Update watchlist and risk dashboard

---

## 15-Minute Quick Price Check Protocol

This is the exact sequence ORACLE runs at the start of every session. It should take 2-3 minutes when everything is clear and 5-10 minutes when something needs attention.

### Step 1: Check All Open Position Prices (60 seconds)
```
WebSearch "META stock price"
WebSearch "AAVE crypto price"
WebSearch "BTC price"
WebSearch "RENDER crypto price"
WebSearch "AMZN stock price"
WebSearch "SUI crypto price"
WebSearch "NVDA stock price"
```
Fill in the "Current" column of the alert levels table above.

### Step 2: Compare to Alert Levels (30 seconds)
For each position:
- Is current price in the warning zone? (within 2% of stop or target)
- Has the stop been breached? (current < stop for longs)
- Has the target been hit? (current > target for longs)

### Step 3: Act on Findings
- **All clear** → Log "All positions clear, no alerts triggered" and move on. Total time: 2 minutes.
- **Warning zone entered** → Investigate why. WebSearch "[ticker] news today". Is this a temporary dip or thesis change? Decide: hold, tighten stop, or prepare to exit.
- **Stop breached** → Mark for exit. Calculate P&L. Alert Presten via all channels. Update risk dashboard.
- **Target hit** → Execute partial or full profit take per exit rules. Log the win. Alert Presten.

### Step 4: Market-Wide Checks (30 seconds)
```
WebSearch "VIX index level"
WebSearch "SPY price today"
WebSearch "BTC 1 hour change"
```
Compare to market-wide alert levels table. Act if any triggered.

### Step 5: Log Session Result
Write a one-line log entry:
```
{{date}} {{time}}: Price check complete. [X] positions checked. [All clear / WARNING: ticker / ALERT: ticker]. VIX: [level]. Next session: [time].
```

---

## Alert Escalation Matrix

Not all alerts are equal. Here is how ORACLE prioritizes when multiple alerts fire:

| Priority | Condition | Response Time | Channels |
|----------|-----------|---------------|----------|
| P0 - Emergency | Position -50%, crash conditions | IMMEDIATE | All three |
| P1 - Critical | Stop breached, VIX 30+ | THIS SESSION | Tweet + Queue (urgent) |
| P2 - Warning | Warning zone, VIX 25 | THIS SESSION | Queue (high) + watchlist update |
| P3 - Watch | Approaching levels, elevated VIX | END OF SESSION | Watchlist update only |
| P4 - Info | Routine price changes, all clear | Log only | Session log |

---

## Weekend and Off-Hours Protocol

Markets close but crypto never sleeps.

### Weekdays 1:00 PM - 4:00 AM PT (after-hours and overnight)
- Crypto moves can still trigger alerts
- Pre-market futures can signal gap risk
- ORACLE runs sessions during this period for crypto monitoring

### Weekends
- Crypto-only monitoring: BTC, SUI, RENDER, AAVE
- Check at minimum twice per day (morning and evening sessions)
- Equity positions are safe (markets closed) but watch for Sunday night futures open

### Holidays
- Same as weekends but also check if any foreign markets are open that could affect positions
- Crypto always trades, always needs monitoring

---

## Maintaining This Engine

Every time a position is opened or closed, this file MUST be updated:
1. **New position** → Add row to alert levels table with entry, stop, target, and calculated warning zone
2. **Position closed** → Remove row from alert levels table, archive to trade journal
3. **Stop adjusted** → Recalculate warning zone (stop x 1.02 for longs)
4. **Target adjusted** → Update target column
5. **HOT watchlist change** → Update the HOT watchlist section

This table is the single source of truth for what ORACLE is monitoring. If it is not in this table, ORACLE is not watching it.
