---
tags: [oracle, trading, correlation, portfolio, risk]
created: 2026-04-21
---

# Correlation Tracker

## The Core Problem

If you're long BTC, ETH, SOL, AMZN, META, NVDA, and QQQ — you think you have seven positions. You actually have **one position: risk-on technology/growth.** When that theme sells off, all seven drop simultaneously, and your "diversification" does nothing.

This is not theory. In each of the three defining crises of the 2000s — the 2008 financial crisis, the 2020 COVID crash, and the 2022 rate-hike drawdown — portfolios that appeared broadly diversified suffered severe, simultaneous losses. Two Sigma's research found pairwise equity correlations surged from ~0.40 before the 2008 crisis to **0.70 during it**. In October 2008, some disparate corporate debt instruments traded at near-perfect 1.0 correlation. During the 2020 COVID crash, Treasuries (traditionally the safe-haven hedge) sold off *alongside* equities — breaking the foundational assumption of the 60/40 portfolio.

**The rule:** Diversification disappears exactly when you need it most. Only true structural hedges (VIX calls, puts, inverse ETFs, cash) survive a correlation spike.

---

## How to Interpret Correlation Numbers

- **0.0:** No relationship. Moves are completely independent.
- **0.3–0.5:** Moderate correlation. Some co-movement in risk-off environments, largely independent otherwise.
- **0.5–0.75:** Significant correlation. These positions will often move together. You have less diversification than you think.
- **0.75–0.90:** High correlation. These are nearly the same bet. Treat them as one position for risk sizing purposes.
- **0.90–1.0:** Near-perfect correlation. Having both is pure redundancy. Pick the one with better risk/reward.
- **Negative correlation (below 0):** True diversification. One tends to rise when the other falls.

---

## Current Correlation Matrix (Update Weekly)

Research basis: CME Group crypto correlation studies (2024–2025), Coinbase Institutional research (Aug 2024), dropstab.com ETH/SOL analysis. Crypto-to-equity correlations range 0.2–0.6 on 1-year rolling basis per CME Group data.

### Risk-On Asset Correlations (normal market conditions)

| | BTC | ETH | SOL | AMZN | META | NVDA | QQQ | RENDER |
|---|---|---|---|---|---|---|---|---|
| **BTC** | 1.0 | | | | | | | |
| **ETH** | 0.88 | 1.0 | | | | | | |
| **SOL** | 0.82 | 0.79 | 1.0 | | | | | |
| **AMZN** | 0.42 | 0.38 | 0.35 | 1.0 | | | | |
| **META** | 0.44 | 0.40 | 0.36 | 0.72 | 1.0 | | | |
| **NVDA** | 0.52 | 0.47 | 0.42 | 0.67 | 0.71 | 1.0 | | |
| **QQQ** | 0.45 | 0.42 | 0.37 | 0.88 | 0.82 | 0.81 | 1.0 | |
| **RENDER** | 0.88 | 0.84 | 0.80 | 0.40 | 0.38 | 0.48 | 0.40 | 1.0 |

**Data notes:**
- BTC-ETH: 90-day rolling correlation of ~0.70–0.88; 1-year figure ~0.85. ETH and SOL "key off Bitcoin" (CME Group, 2025).
- ETH-SOL 90-day: ~0.70–0.79 (dropstab research, 2024–2025)
- SOL average correlation to BTC in H1 2024: 0.71 (higher than ETH's 0.61 in the same period)
- Crypto-to-Nasdaq: +0.2 to +0.6 on 1-year rolling basis (CME Group); NVDA-BTC correlation was at a multi-year high in March 2024
- AMZN-QQQ and META-QQQ: Both are top-weight Nasdaq constituents — correlation naturally high
- RENDER: AI/crypto hybrid; trades as a high-beta altcoin, correlates strongly with BTC/ETH

### Crisis Correlation (what it becomes in a crash)

During systemic risk events (all correlations spike toward 1.0):

| | BTC | ETH | SOL | AMZN | META | NVDA | QQQ |
|---|---|---|---|---|---|---|---|
| **BTC** | 1.0 | | | | | | |
| **ETH** | 0.95+ | 1.0 | | | | | |
| **SOL** | 0.93+ | 0.90+ | 1.0 | | | | |
| **AMZN** | 0.70+ | 0.68+ | 0.65+ | 1.0 | | | |
| **META** | 0.72+ | 0.69+ | 0.66+ | 0.85+ | 1.0 | | |
| **NVDA** | 0.78+ | 0.75+ | 0.70+ | 0.80+ | 0.82+ | 1.0 | |
| **QQQ** | 0.72+ | 0.68+ | 0.65+ | 0.92+ | 0.88+ | 0.88+ | 1.0 |

**This is not speculation — this is what happened in 2008, 2020, and 2022.** Just three principal components explained 90% of variation across four major asset classes during 2008 (vs. 70% in normal markets), per Two Sigma research. In the 2020 COVID crash, correlations across nearly every asset class turned strongly positive simultaneously.

---

## The Effective Number of Independent Positions Formula

The illusion of diversification is quantifiable.

**Formula:**
```
N_eff = N / (1 + (N - 1) × ρ_avg)
```
Where N = number of positions and ρ_avg = average pairwise correlation.

**Examples with real numbers:**

| Portfolio | N | Avg Correlation | N_eff | Reality |
|-----------|---|----------------|-------|---------|
| BTC + ETH + SOL | 3 | 0.83 | 3 / (1 + 2×0.83) = **1.11** | You have 1 position |
| BTC + ETH + SOL + NVDA + QQQ | 5 | 0.65 | 5 / (1 + 4×0.65) = **1.47** | You have 1.5 positions |
| Full ORACLE portfolio (8 assets, above matrix) | 8 | ~0.55 | 8 / (1 + 7×0.55) = **1.98** | You have ~2 positions |
| Truly diversified (long + short, crypto + bonds + commodities) | 6 | 0.20 | 6 / (1 + 5×0.20) = **3.00** | You have 3 real positions |

**Use this formula every time you add a position.** Adding a 6th highly correlated name does not reduce your risk — it concentrates it.

---

## How to Actually Diversify

### True Diversifiers (negative or near-zero correlation to risk-on assets)

| Asset | Correlation to BTC/Tech | Use Case |
|-------|------------------------|---------|
| **Gold (GLD)** | 0.0 to +0.2 (varies) | Mild inflation/uncertainty hedge |
| **VIX calls / UVIX** | -0.6 to -0.9 in crashes | Direct crash protection; most effective hedge |
| **Long bonds (TLT)** | Historically negative; unreliable in 2022 | Rate-driven protection; context-dependent |
| **Energy (XLE)** | -0.1 to +0.2 | Different economic driver; true diversifier in inflation |
| **Short QQQ or SPY** | -0.9 to -1.0 | Direct market hedge; reduces beta |
| **Put options on portfolio holdings** | -0.8 to -1.0 | Defined-risk crash protection |
| **Cash (USD)** | 0.0 | The ultimate hedge — no return, but no correlation |
| **Inverse ETFs (SQQQ, SPXS)** | -0.8 to -1.0 | Active crash hedging tool |

### Pairs Trades (sector-neutral positions)

Long one name, short another in the same sector. Correlation within a sector is high (0.7+), so the pair isolates the relative performance rather than taking a market bet:
- Long NVDA / Short AMD (bet on NVDA outperforming, not on tech going up)
- Long BTC / Short ETH (bet on BTC dominance increasing)
- Long XLU (utilities) / Short QQQ (risk-off vs. risk-on)
- Long GLD / Short SPY (gold vs. equities rotation)

The pairs trade eliminates most market beta. Your P&L depends on relative performance, not direction.

---

## Correlation Regime Detection

Correlations are not static. They shift with market regimes.

### Regime 1: Risk-On (Normal Bull Market)
- Crypto-to-equity correlations: 0.3–0.5
- Within-crypto correlations: 0.7–0.9
- VIX < 20
- Bonds and stocks move independently or inversely
- **Action:** Standard diversification across asset classes provides real benefit

### Regime 2: Risk-Off (Late Cycle, Rising Rates, Uncertainty)
- Crypto-to-equity correlations: 0.5–0.7
- Within-crypto correlations: 0.85–0.95
- VIX: 20–35
- Stocks and bonds can sell simultaneously
- **Action:** Reduce crypto allocation, increase cash, add sector diversifiers (energy, gold)

### Regime 3: Crisis (Systemic Risk, Crash)
- ALL risk-on asset correlations → 0.85–1.0
- VIX > 35
- Traditional diversification fails completely
- **Only protection:** VIX calls, puts, cash, inverse ETFs
- **Action:** Activate crash playbook (see risk-dashboard.md), raise cash to 50%+, add VIX calls

### How to Detect Regime Changes
Watch these signals for regime shifts:
1. **VIX crossing above 25:** Risk-off beginning. Tighten stops, reduce size.
2. **Credit spreads widening (HYG falling):** Systemic stress signal. Reduce equities.
3. **BTC-equity correlation rising above 0.6 on 30-day basis:** Risk assets becoming more correlated — less diversification benefit.
4. **Gold and stocks declining together:** Late-stage panic; forced selling of everything. Maximum correlation event beginning.
5. **Dollar (DXY) sharply rising:** Global risk-off; USD being hoarded; all risk assets under pressure.

---

## Weekly Correlation Check

Complete every Sunday or Monday before the week opens:

```
ORACLE WEEKLY CORRELATION CHECK
================================
Date: ____

CORRELATION MATRIX UPDATE
--------------------------
Highest correlated pair: ____ / ____ (correlation: ____)
Lowest correlated pair: ____ / ____ (correlation: ____)
Any correlation significantly changed from last week? ____

PORTFOLIO CALCULATION
--------------------------
Number of positions: ____
Average pairwise correlation: ____
Effective independent positions (N_eff): ____

REGIME ASSESSMENT
--------------------------
Current VIX level: ____
VIX trend: RISING / FALLING / FLAT
Credit spreads (HYG direction): TIGHTENING / WIDENING / FLAT
BTC-Equity 30-day correlation: ____

Current regime: RISK-ON / RISK-OFF / CRISIS
Regime trend: IMPROVING / DETERIORATING / STABLE

PORTFOLIO ASSESSMENT
--------------------------
Portfolio is: DIVERSIFIED / CONCENTRATED / DANGEROUSLY CORRELATED

ACTIONS NEEDED
--------------------------
[ ] Reduce position in: ____
[ ] Add hedge: ____
[ ] Pairs trade opportunity: ____
[ ] Raise cash to: ____% (current: ____%)

NOTES
--------------------------
____
```

---

## The One Rule That Overrides Everything

When market-wide correlation spikes to crisis levels, **position size and diversification do not protect you.** The only things that protect you:

1. **Cash held before the crash** — you cannot raise it during the crash without locking in losses
2. **Puts bought before IV spikes** — VIX calls at 15 are 5x cheaper than VIX calls at 35
3. **Short positions or inverse ETFs already in place** — not initiated in panic

This is why the correlation tracker exists: to catch the regime change *before* it becomes a crisis. Rising correlations + rising VIX + widening credit spreads = the early warning system. Act on it early, not after.

The cost of maintaining small hedges in calm markets (maybe -1 to -2% annual drag) is vastly less than the cost of a -40% drawdown with no protection.
