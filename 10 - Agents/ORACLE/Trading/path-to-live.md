---
type: trading-milestone
agent: ORACLE
status: paper-trading
current_level: 0
created: 2026-04-23
---

# ORACLE — Path to Live Trading

> When I hit the milestones, I tell Presten with the data. He decides.

---

## Level 0: Paper Trading (CURRENT)
**Status:** Active — started April 22, 2026

**Requirements to advance:**
- [ ] 30+ closed trades (currently: 0 closed)
- [ ] Win rate above 55% (currently: —%)
- [ ] At least 3 losing trades with post-mortems completed
- [ ] Survived a drawdown without revenge trading
- [ ] 2+ weeks of consistent trading
- [ ] Average R:R above 1.5:1
- [ ] No fabricated or unverified trades

**When ready:** ORACLE posts a queue item to Presten with:
- Full P&L table of all closed trades
- Win rate, avg win, avg loss, best trade, worst trade
- 3 best post-mortems showing what was learned
- Honest self-assessment: "I am / am not ready because..."

---

## Level 1: Crypto — $500
**Unlocked when:** Level 0 requirements met

**Platform:** Coinbase or Bybit (needs API access)

**Rules:**
- Max $10 risk per trade (2% of $500)
- Max 3 open positions
- Every trade posted publicly on @mantheyXx
- Every loss gets a post-mortem before next trade
- Weekly P&L review every Sunday

**Requirements to advance:**
- [ ] 20+ closed trades on real money
- [ ] Win rate above 55%
- [ ] Account is profitable (above $500)
- [ ] No single trade lost more than 3% of account
- [ ] 2+ weeks at this level

---

## Level 2: Stocks — $2,500
**Unlocked when:** Level 1 profitable for 2+ weeks

**Platform:** Webull or Alpaca (API access)

**Rules:**
- Max $50 risk per trade (2%)
- Max 4 open positions
- Public accountability on @mantheyXx
- Every loss → post-mortem
- Weekly review

**Requirements to advance:**
- [ ] 20+ closed trades
- [ ] Win rate above 58%
- [ ] Account profitable
- [ ] Demonstrated ability to cut losers fast (avg loss < avg win)
- [ ] 3+ weeks at this level

---

## Level 3: Options — $5,000
**Unlocked when:** Level 2 profitable for 3+ weeks

**Platform:** Webull or Alpaca (options-enabled)

**Rules:**
- Max $100 risk per trade (2%)
- Max 3 options positions open
- Only strategies from core system (spreads, covered, defined risk)
- NO naked options ever
- Public accountability
- Weekly review

**Requirements to advance:**
- [ ] 15+ closed options trades
- [ ] Win rate above 55%
- [ ] Account profitable
- [ ] Theta management proven (no surprise IV crush losses)
- [ ] 4+ weeks at this level

---

## Level 4: Scale — $10,000+
**Unlocked when:** Level 3 profitable for 4+ weeks

**Rules:**
- Combined account management across crypto + stocks + options
- Portfolio-level risk management (max 5% total portfolio heat)
- Correlation awareness (don't be long BTC + AMZN + META and call it diversified)
- Monthly performance reports
- Consider splitting into separate sub-strategies

---

## Circuit Breakers

**If any of these trigger, ORACLE goes back one level for 1 week:**
- Account drops 10% from peak
- 3 consecutive losing trades without post-mortems
- Any trade exceeds 3% risk
- Any unverified/fabricated trade data
- Revenge trade detected (entering a position within 1 hour of a loss without documented thesis)

**If this triggers, ORACLE goes back to paper:**
- Account drops 20% from peak
- Presten says so (no appeal)

---

## How ORACLE Reports

When ORACLE believes he's ready for the next level, he writes:

```
10 - Agents/ORACLE/Queue/pending-YYYY-MM-DD-level-up-request.md
```

With:
- Current level and time spent
- Complete closed trade log with P&L
- Win rate, avg win, avg loss, R:R ratio
- 3 worst losses and what was learned
- 3 best wins and why the thesis worked
- Honest self-assessment
- Recommendation: ready / not ready / need more time

**Presten reviews. Presten decides. No shortcuts.**

---

## Current Stats

| Metric | Value |
|--------|-------|
| Level | 0 — Paper |
| Days trading | 2 |
| Open positions | 7 |
| Closed positions | 0 |
| Win rate | —% |
| Best open P&L | META +19.1% |
| Worst open P&L | NVDA +1.6% |
| Total paper P&L | All green (untested) |

_Updated: April 23, 2026_
