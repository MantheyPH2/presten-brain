---
type: trading-tool
agent: ORACLE
title: Automated P&L Calculator
created: 2026-04-21
last_updated: "{{date}} {{time}}"
---

# Automated P&L Calculator

**ORACLE runs this every session. Not occasionally. Not when it feels relevant. EVERY session. The numbers do not lie, and ORACLE does not guess — he calculates.**

---

## Current Session P&L Snapshot

```
DATE: ____________
TIME: ____________
ACCOUNT VALUE: $____________

POSITION TABLE:
| # | Ticker | Dir | Entry | Current | Shares/Contracts | Cost Basis | Market Value | P&L $ | P&L % | Weight % | Days Held |
|---|--------|-----|-------|---------|-----------------|-----------|-------------|-------|-------|----------|-----------|
| 1 | META   | L   | $565  |         |                 |           |             |       |       |          |           |
| 2 | AAVE   | L   | $88   |         |                 |           |             |       |       |          |           |
| 3 | BTC    | L   | $74,000 |       |                 |           |             |       |       |          |           |
| 4 | RENDER | L   | $1.72 |         |                 |           |             |       |       |          |           |
| 5 | AMZN   | L   | $248  |         |                 |           |             |       |       |          |           |
| 6 | SUI    | L   | $0.93 |         |                 |           |             |       |       |          |           |
| 7 | NVDA   | L   | $198  |         |                 |           |             |       |       |          |           |
|---|--------|-----|-------|---------|-----------------|-----------|-------------|-------|-------|----------|-----------|
| TOTAL |    |     |       |         |                 | $______   | $______     |$_____ | ____% | 100%     |           |

CASH REMAINING: $____________
TOTAL ACCOUNT: $____________ (positions + cash)

DAILY CHANGE:   $________ (______%)
WEEKLY CHANGE:  $________ (______%)
MTD CHANGE:     $________ (______%)
ALL-TIME P&L:   $________ (______%)
```

---

## Auto-Calculation Rules

ORACLE executes these steps IN ORDER every session. No shortcuts.

### Step 1: Get Current Prices
```
WebSearch "META stock price"
WebSearch "AAVE price"
WebSearch "BTC price"
WebSearch "RENDER price"  
WebSearch "AMZN stock price"
WebSearch "SUI price"
WebSearch "NVDA stock price"
```
Fill in the "Current" column for every position.

### Step 2: Calculate Individual P&L
For each position:
```
P&L % = (Current Price - Entry Price) / Entry Price x 100
P&L $ = (Current Price - Entry Price) x Number of Shares/Contracts
```

For short positions (when applicable):
```
P&L % = (Entry Price - Current Price) / Entry Price x 100
P&L $ = (Entry Price - Current Price) x Number of Shares/Contracts
```

### Step 3: Calculate Position Weights
```
Position Value = Current Price x Shares/Contracts
Position Weight = Position Value / Total Portfolio Value x 100
```

Sanity checks:
- All weights must sum to approximately 100% (remaining is cash)
- Any single position above 30% = concentrated risk warning
- Any single position above 50% = IMMEDIATELY flag for Presten

### Step 4: Calculate Portfolio Totals
```
Total Market Value = Sum of all position values
Total P&L $ = Sum of all individual P&L $
Total P&L % = Total P&L $ / Starting Capital x 100
Daily Change = Today's total value - Yesterday's total value
Weekly Change = Today's total value - Last Monday's total value
```

### Step 5: Update Running Statistics
```
WIN RATE:        ____% (closed winning trades / total closed trades)
TOTAL WINS:      ____
TOTAL LOSSES:    ____
BEST TRADE:      [ticker] +____% ($____)
WORST TRADE:     [ticker] -____% ($____)
AVERAGE WIN:     +____%
AVERAGE LOSS:    -____%
PROFIT FACTOR:   ____ (sum of all gains / sum of all losses; >1.5 is good, >2.0 is excellent)
EXPECTANCY:      $____ per trade (avg win x win rate - avg loss x loss rate)
SHARPE ESTIMATE: ____ (avg return / std dev of returns; >1.0 is acceptable, >2.0 is excellent)
MAX DRAWDOWN:    -____% (largest peak-to-trough decline)
CURRENT DRAWDOWN:-____% (current distance from all-time high)
```

### Step 6: Cross-Reference
1. Update [[risk-dashboard]] with current numbers
2. Update [[tiered-watchlist]] with current prices and P&L
3. Update [[price-alert-engine]] with current prices

### Step 7: Flag Anomalies
If ANY of these are true, investigate immediately:
- Any position changed more than 3% since last check → `WebSearch "[ticker] news today"`
- Portfolio total changed more than 5% since last check → market-wide event, check VIX
- Any position weight shifted more than 10 percentage points → rebalance discussion
- Win rate dropped below 40% → review strategy, not just bad luck
- Profit factor dropped below 1.0 → STOP TRADING and review everything

---

## Position Sizing Calculator

Run this BEFORE every new trade. This is not optional. This is the gate between "I want to trade this" and "I am allowed to trade this."

### Input Variables
```
ACCOUNT VALUE:     $__________ (from latest P&L snapshot above)
CONVICTION:        ___/5 (from pre-trade checklist)
CURRENT MODE:      ATTACK / AGGRESSIVE / BALANCED / PROTECTIVE / FORTRESS
```

### Conviction-Based Risk Table

| Conviction | Max Risk % | Description |
|-----------|-----------|-------------|
| 5/5 | 2.0% | Highest conviction. Multiple confluence factors. Strong catalyst. Clear edge. |
| 4/5 | 1.5% | High conviction. Most factors align. Minor uncertainty on one element. |
| 3/5 | 1.0% | Moderate. Decent setup but missing one key confirmation. Standard size. |
| 2/5 | 0.5% | Low conviction. Speculative. Small size only. Must have defined catalyst. |
| 1/5 | 0.25% | Lottery ticket. Almost certainly wrong but asymmetric payoff justifies tiny bet. |

### Mode Multiplier

| Mode | Multiplier | When |
|------|-----------|------|
| ATTACK | 2.0x | Account under $5K. Need growth. Accept higher risk. |
| AGGRESSIVE | 1.5x | Account $5K-$25K. Growing but not reckless. |
| BALANCED | 1.0x | Account $25K-$100K. Standard risk management. |
| PROTECTIVE | 0.5x | Drawdown mode. Recent losses. Rebuild confidence. |
| FORTRESS | 0.25x | Max drawdown hit. Minimum position sizes until winning resumes. |

### Calculation

```
STEP 1: MAX RISK %
   Base risk (from conviction table):    ____%
   Mode multiplier:                      ____x
   Adjusted max risk:                    ____% (base x multiplier)

STEP 2: MAX DOLLAR RISK
   Account value:                        $________
   Adjusted max risk %:                  ____%
   Max dollar risk:                      $________ (account x risk %)

STEP 3: POSITION SIZE
   Entry price:                          $________
   Stop price:                           $________
   Risk per share:                       $________ (entry - stop for longs; stop - entry for shorts)
   Position size (shares):               ________ (max dollar risk / risk per share)
   Position value:                       $________ (shares x entry price)

STEP 4: PORTFOLIO WEIGHT CHECK
   Position value:                       $________
   Total account value:                  $________
   Portfolio weight:                     ____% (position value / account value x 100)
```

### Sanity Checks (ALL must pass)
- [ ] Portfolio weight is under 30% for a single position (unless ATTACK mode on a 5/5 conviction play)
- [ ] Total portfolio heat (all open risk) stays under 6% after this trade
- [ ] There is enough cash remaining to cover the QQQ hedge if needed
- [ ] This trade does not add a 4th correlated position in the same risk-on cluster without a hedge
- [ ] The risk/reward ratio is at least 1.5:1

**If any check fails: REDUCE SIZE until all pass, or DO NOT TRADE.**

---

## Closed Trades Log

Every closed trade gets logged here with full P&L. This is the source of truth for win rate and performance statistics.

| # | Ticker | Direction | Entry | Exit | Shares | P&L $ | P&L % | Days Held | Reason for Exit | Win/Loss |
|---|--------|-----------|-------|------|--------|-------|-------|-----------|----------------|----------|
| 1 | | | | | | | | | | |
| 2 | | | | | | | | | | |
| 3 | | | | | | | | | | |

---

## Weekly P&L Summary

| Week Starting | Starting Value | Ending Value | Change $ | Change % | Trades Made | Win Rate |
|--------------|---------------|-------------|----------|----------|-------------|----------|
| | | | | | | |
| | | | | | | |

---

## Monthly P&L Summary

| Month | Starting Value | Ending Value | Change $ | Change % | Total Trades | Win Rate | Best Trade | Worst Trade |
|-------|---------------|-------------|----------|----------|-------------|----------|-----------|-------------|
| Apr 2026 | ~$1,000 | | | | | | | |
| May 2026 | | | | | | | | |
| Jun 2026 | | | | | | | | |

---

## Equity Curve Data Points

Track the account value at the end of each week to build an equity curve over time. A rising equity curve with controlled drawdowns is the goal. A jagged curve with deep drops means risk management needs work.

| Date | Account Value | Drawdown from Peak | Notes |
|------|-------------|-------------------|-------|
| | | | |
| | | | |

When this table has 12+ weeks of data, ORACLE can calculate:
- Rolling 4-week return
- Maximum drawdown duration (how long from peak to recovery)
- Sharpe ratio with real data (not estimates)
- Whether the account is actually growing or just churning
