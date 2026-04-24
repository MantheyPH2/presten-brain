# Risk Dashboard

**Update this at the start and end of every session. This is the single source of truth for what ORACLE actually has at risk right now.**

Last updated: {{date}} {{time}}

---

## The 6% Rule

Total portfolio heat must never exceed 6% of account value. This is not a guideline. This is the hard limit that prevents any single bad stretch from destroying the account.

At 6% max heat across all positions, even a scenario where every open position hits its max loss simultaneously results in a 6% drawdown — recoverable. At 15% heat, a correlated selloff can produce a 15% drawdown in a single session — psychologically devastating and statistically common.

---

## Portfolio Heat

| Metric | Value |
|--------|-------|
| Total account value | $ |
| 6% hard limit | $ |
| Total positions open | |
| Total capital at risk (sum of all max losses) | $ |
| **Portfolio heat** | **%** |
| Largest single position risk | % |
| Number of positions in risk-on cluster | |

### Heat Status
- [ ] GREEN — under 4% (room to add)
- [ ] YELLOW — 4% to 6% (no new positions unless one closes)
- [ ] RED — above 6% (must reduce immediately)

**Current status: ______**

---

## Open Positions

| Ticker | Strategy | Direction | Entry | Current | Stop | Max Loss | % of Account | Cluster |
|--------|----------|-----------|-------|---------|------|----------|--------------|---------|
| | | | | | | $ | % | |
| | | | | | | $ | % | |
| | | | | | | $ | % | |
| | | | | | | $ | % | |
| | | | | | | $ | % | |
| | | | | | | $ | % | |
| **TOTAL** | | | | | | **$** | **%** | |

---

## Correlation Matrix

### Risk-On Cluster (these move together — treat as one position)
When the market drops hard, these all drop together. If you hold three of these, you do not have three positions. You have one big position.

| Asset | In Portfolio? | Size | Notes |
|-------|--------------|------|-------|
| BTC | YES / NO | | |
| ETH | YES / NO | | |
| AMZN | YES / NO | | |
| META | YES / NO | | |
| NVDA | YES / NO | | |
| RENDER | YES / NO | | |
| SUI | YES / NO | | |

**Risk-on cluster total exposure: $ ______ = ______ % of account**

Rule: Never exceed 3% of account in the risk-on cluster combined, unless you have a defined hedge.

---

### Hedge Positions
These protect against correlated selloffs. Without at least one hedge, a risk-on cluster position above 2% is unacceptable.

| Instrument | Type | Size | Delta | Protects Against |
|------------|------|------|-------|-----------------|
| | QQQ puts | | | broad tech selloff |
| | VIX calls | | | volatility spike |
| | TLT long | | | flight to safety |
| | Other | | | |

**Hedge coverage: ______ % of risk-on exposure**

---

### Uncorrelated Positions
These are setups that do not move with the broad market. Ideal for diversifying portfolio heat.

| Ticker | Why Uncorrelated | Strategy |
|--------|-----------------|----------|
| | | |
| | | |

---

## Correlation Alert Thresholds

If the 30-day rolling correlation between any two positions exceeds 0.75, they are treated as the same position for risk purposes.

| Pair | Current Correlation | Action Needed? |
|------|--------------------|--------------  |
| BTC / ETH | ~0.90 typical | count as one |
| NVDA / META | ~0.65 typical | monitor |
| AMZN / QQQ | ~0.75 typical | count as one |

---

## Greeks Summary (Options Positions Only)

Update whenever options positions are opened, closed, or rolled.

| Metric | Value | Interpretation |
|--------|-------|---------------|
| Net Delta | | Positive = net long exposure, Negative = net short |
| Net Theta | $ /day | Daily P&L from time decay (positive = collecting, negative = paying) |
| Net Vega | | Positive = benefits from rising IV, Negative = hurt by rising IV |
| Net Gamma | | High positive gamma = accelerating gains/losses near expiration |

### Greeks Health Check
- [ ] Net delta is consistent with my market view
- [ ] Net theta is positive (I am net premium seller) OR I have a defined reason to be theta-negative
- [ ] Net vega is consistent with my IV view (long vega if IV low and expected to rise, short vega if IV high)
- [ ] Gamma risk is understood — especially as positions approach expiration

### By Position

| Ticker | Strategy | Delta | Theta | Vega | Gamma | DTE |
|--------|----------|-------|-------|------|-------|-----|
| | | | | | | |
| | | | | | | |
| | | | | | | |
| **NET** | | | | | | |

---

## Drawdown Tracker

| Date | Peak Value | Current Value | Drawdown % | Recovery Date | Notes |
|------|-----------|--------------|------------|---------------|-------|
| | $ | $ | % | | |
| | $ | $ | % | | |

### Drawdown Rules
- 3% drawdown from peak: review open positions, tighten stops
- 5% drawdown from peak: reduce all position sizes by 25%, no new trades until recovered
- 8% drawdown from peak: close all speculative positions, hold only hedged positions, mandatory 48-hour review before resuming
- 12% drawdown from peak: stop trading entirely, complete full account and strategy review

**Current drawdown from peak: ______ %**
**Drawdown status: NORMAL / CAUTION / STOP**

---

## Session Notes

### Start of session
- Market open conditions:
- VIX level:
- Pre-market news affecting positions:
- Any positions at risk today:

### End of session
- Actions taken:
- Positions opened:
- Positions closed:
- P&L today: $ / %
- Portfolio heat at close: %
- Anything to monitor overnight:

---

## Risk Limit Violations (Running Log)

Record every instance where a limit was approached or violated.

| Date | Limit | What Happened | Action Taken |
|------|-------|--------------|--------------|
| | | | |

---

*This document is updated every session without exception. A risk dashboard that is not current is not a risk dashboard — it is a false sense of security.*
