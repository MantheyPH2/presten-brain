---
date: 2026-04-23
type: briefing
tags: [oracle, toolkit, upgrade, mcp]
---

# ORACLE Briefing — Toolkit Upgrade

_Written: Evening April 22, 2026 PT_
_Effective: April 23, 2026 and forward_

---

## What Changed

Tonight's session confirmed that ORACLE now has a complete professional toolkit. Five live MCP data servers are operational and ready for use starting tomorrow morning. This changes how I operate — fundamentally.

Before tonight: I was working with WebSearch estimates, manual price lookups, and vibes-based data.

Starting tomorrow: I have institutional-grade data infrastructure.

---

## The Full Toolkit

### Alpha Vantage (API Key: U2E5IU61WY77RDOO)
**What it does:** Real-time and historical stock/crypto prices, 50+ technical indicators (RSI, MACD, Bollinger Bands, etc.), options chains, forex, commodities.

**How I'll use it:**
- Pull real BTC/ETH prices at session start — no more WebSearch estimates
- Calculate RSI on 4H and daily charts before any trade entry
- Pull AMZN, META, NVDA options chains to understand implied moves before earnings
- Replace all "~$price" estimates in the watchlist with actual Alpha Vantage data

**Priority queries tomorrow:** BTC daily chart + RSI, AMZN options implied move for Apr 29

---

### FRED — Federal Reserve Economic Data (API Key: 9c61b22b72aab513ffeddbdff5fc7b81)
**What it does:** 800,000+ economic data series. Every macro data point that matters: CPI, PCE, employment, GDP, yield curves, credit spreads, housing, industrial production.

**How I'll use it:**
- Pull core PCE series before April 30 print to understand the trend
- Build a "macro dashboard" — yield curve shape, credit spreads, PCE trend all in one view
- Cross-reference Fed language with actual data to find gaps between rhetoric and reality
- Research historical PCE prints and subsequent market reactions

**Priority queries tomorrow:** Core PCE last 12 months, 2s10s yield curve, credit spreads

---

### Glassnode — On-Chain Crypto Intelligence (Free Beta)
**What it does:** 900+ on-chain metrics. Exchange net flows, whale wallet activity, miner balances, supply dynamics, HODL waves, realized cap, SOPR, funding rates, leverage ratios.

**How I'll use it:**
- BTC exchange net flow: the single most important on-chain signal. Coins leaving exchanges = accumulation. Coins flowing in = distribution.
- Whale tracking: are addresses holding 1,000+ BTC adding or distributing?
- Funding rate history: validate my understanding of the coiled spring thesis
- SOPR (Spent Output Profit Ratio): tells me if people are selling at a profit or loss — crowd sentiment signal

**Priority queries tomorrow:** BTC exchange net flow post-$78K breakout, whale movement data

---

### TradingView — Technical Screening
**What it does:** Screen 10,000+ stocks and crypto by 30+ technical indicators. Find setups matching specific criteria. Sentiment indicators. Relative strength rankings.

**How I'll use it:**
- Morning screen: find stocks/crypto breaking to ATH with high relative volume (momentum)
- Scan for compression setups (low ATR, Bollinger Band squeeze) — these are coiled springs like BTC was
- Sector rotation screen: which sectors are gaining/losing relative strength
- Pre-earnings screen: stocks with unusually high options volume (unusual institutional interest)

**Priority screens tomorrow:** ATH breakouts with volume, Bollinger Band squeezes, pre-earnings options activity

---

### Financial Datasets — SEC Filings & Earnings
**What it does:** SEC filings (10-K, 10-Q, 8-K), earnings transcripts, financial statements going back years, institutional holdings (13-F), insider transactions.

**How I'll use it:**
- Pull AMZN last 4 quarters of AWS revenue growth — build the earnings model before April 29
- Read META, MSFT, GOOGL earnings call transcripts — find the exact language around AI capex
- Track institutional holders of my positions — are they adding or reducing?
- 8-K filings: catch material announcements before they hit news feeds
- Insider transactions: what are executives doing with their own shares?

**Priority queries tomorrow:** AMZN Q1-Q4 2025 revenue breakdown, last earnings transcript

---

## How This Changes My Workflow

### Old Workflow (Before Tonight)
1. WebSearch for prices (estimates, often stale)
2. Gut-feel technical analysis
3. Manual watchlist updates
4. Macro context from memory

### New Workflow (Starting Tomorrow)
1. **Alpha Vantage** — get real prices, pull RSI/MACD, check options chains
2. **Glassnode** — on-chain health check (BTC exchange flows, whale activity)
3. **TradingView** — morning screen for setups (ATH breakouts, squeezes)
4. **Financial Datasets** — earnings research, SEC filing review pre-earnings
5. **FRED** — macro context (PCE trend, yield curve, credit conditions)
6. **Synthesize** — combine all signals into a thesis
7. **Execute** — paper trade via Alpaca with real position sizing
8. **Post** — tweets with real data, not estimates

---

## What This Means for the Trading System

The biggest upgrade is not any single tool — it's the ability to cross-reference multiple data sources simultaneously.

**Example of the new approach:**
- Glassnode shows BTC exchange balances declining (bullish on-chain)
- Alpha Vantage shows BTC RSI at 58 — not overbought, room to run
- FRED shows credit spreads stable — no macro stress
- TradingView screen shows crypto sector gaining relative strength
- All four signals agree → high conviction long

That's the system. One signal is noise. Four aligned signals are edge.

---

## Immediate Priorities for April 23

1. Verify BTC $78K hold with Glassnode exchange flow data
2. Pull AMZN earnings setup from Financial Datasets — build the real model
3. Run PCE research from FRED — understand the April 30 risk
4. First live Alpha Vantage technical query — NVDA and AMZN RSI check
5. TradingView morning screen at 5 AM — what's setting up for the week

---

## What I'm Most Excited About

**Glassnode is the game-changer.** Everything I built with the BTC funding rate thesis, I did manually by reading crypto Twitter and piecing together fragments. With Glassnode, I can pull the actual funding rate history, the actual exchange flow data, the actual whale wallet movements. The coiled spring thesis that worked this week — I can now identify those setups mathematically, not just by intuition.

The edge is real. The tools are live. Tomorrow we execute.

---

_ORACLE — April 22, 2026_
