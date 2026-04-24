# ORACLE Pre-Market Scanner

> **Run daily 5:00-6:30 AM PT | Before every trading session**
> The first 90 minutes before market open determine 80% of the day's edge. This scanner is non-negotiable.

---

## What to Scan

### 1. Futures Direction (5:00 AM PT)
The single most important pre-market signal. Futures tell you the market's bias before a single share trades.

| Instrument | Ticker | Where to Check |
|-----------|--------|---------------|
| S&P 500 futures | /ES, SPY | investing.com/indices/us-spx-500-futures |
| Nasdaq 100 futures | /NQ, QQQ | investing.com/indices/nq-100-futures |
| Dow futures | /YM, DIA | investing.com/indices/us-30-futures |
| Russell 2000 futures | /RTY, IWM | investing.com/indices/smallcap-2000-futures |
| VIX | VIX | cboe.com |

**Current context (April 2026):** S&P 500 at ~7,108 | Dow at ~49,310 | Nasdaq at ~24,438 | VIX at ~18.92. Oil above $105/bbl on Iran war uncertainty. Intel surged 19% after-hours on earnings beat.

### 2. Pre-Market Movers (5:15 AM PT)
Stocks moving 3%+ before the bell. These are where the action is.

**What causes pre-market gaps:**
- Earnings beats/misses (most common)
- FDA approvals/rejections
- M&A announcements
- Analyst upgrades/downgrades
- Macro data surprises
- Overnight news, geopolitical events

**Where to find them:**
- `WebSearch "pre-market movers today"`
- [MarketChameleon Pre-Market](https://marketchameleon.com/Reports/PremarketTrading)
- [Benzinga Pre-Market](https://www.benzinga.com/premarket)
- [StockAnalysis Pre-Market](https://stockanalysis.com/markets/premarket/)
- [CNN Pre-Markets](https://www.cnn.com/markets/premarkets)
- [TradingView Pre-Market Gainers](https://www.tradingview.com/markets/stocks-usa/market-movers-pre-market-gainers/)

**Recent example movers (April 2026):**
- AUUD gapped up 77% on AI data center patent news
- QS gapped up 34% on Q1 earnings beat + Eagle Line production
- AKAN up 34% on cannabis reclassification speculation
- KREF down 22% on commercial real estate weakness

### 3. Overnight Crypto Moves (5:00 AM PT)
Crypto trades 24/7. Overnight moves signal risk appetite.

| Asset | Where to Check | Signal Threshold |
|-------|---------------|-----------------|
| BTC | coingecko.com, coinmarketcap.com | +/- 3% = notable, +/- 5% = significant |
| ETH | coingecko.com | +/- 4% = notable, +/- 7% = significant |
| SOL | coingecko.com | +/- 5% = notable, +/- 10% = significant |
| BTC Dominance | tradingview.com | Rising = risk-off within crypto |
| Total Crypto Market Cap | coingecko.com | Direction of overall crypto sentiment |

**Current context (April 2026):** BTC at ~$77,900 | ETH correcting ~2% | SOL at ~$85.74 down 2.5% | BTC dominance at 58.1% (capital flowing to BTC from alts) | Total market cap ~$2.68T | Fear index at 46 (neutral-fear)

### 4. Asia/Europe Close (5:00 AM PT)
International markets close before US opens. They front-run or confirm US direction.

| Market | Hours (PT) | What to Watch |
|--------|-----------|--------------|
| Nikkei 225 (Japan) | Closes 1:00 AM PT | BOJ policy, yen strength, tech sentiment |
| Shanghai Composite (China) | Closes 12:00 AM PT | China stimulus, trade tensions, property sector |
| Hang Seng (Hong Kong) | Closes 1:00 AM PT | China tech, capital flows |
| FTSE 100 (UK) | Closes 8:30 AM PT | Still open at scan time — watch direction |
| DAX (Germany) | Closes 8:30 AM PT | European manufacturing, ECB policy |
| Euro Stoxx 50 | Closes 8:30 AM PT | Broad European sentiment |

**WebSearch queries:** `"Asia markets close today"` + `"Europe stock markets today"`

### 5. Pre-Market Volume Leaders (5:30 AM PT)
Unusual volume without a clear catalyst = something is happening that isn't public yet.

**Rules:**
- Pre-market volume > 5x average daily volume = investigate immediately
- High volume + no news = potential leak (M&A, FDA, insider activity)
- High volume + earnings = expected, focus on the direction
- Track volume relative to the stock's normal pre-market activity

### 6. Earnings Before the Bell (5:15 AM PT)
Who reported pre-market? This drives the biggest gaps.

**Check:**
- [Earnings Whispers Calendar](https://www.earningswhispers.com/calendar)
- [The Whisper Number Calendar](https://thewhispernumber.com/calendar)
- `WebSearch "earnings today pre-market"`
- Did they beat/miss consensus AND whisper? (See [[earnings-whisper-system]])
- What was the implied move vs actual move?
- Any guidance changes?

### 7. Economic Data Releases (5:30 AM PT)
Most major data drops at 8:30 AM ET (5:30 AM PT). Know what's coming.

| Data | Typical Release | Market Impact |
|------|----------------|---------------|
| Non-Farm Payrolls | First Friday monthly, 8:30 AM ET | HIGH — moves everything |
| CPI (Inflation) | ~12th of month, 8:30 AM ET | HIGH — Fed policy driver |
| PPI (Producer Prices) | ~14th of month, 8:30 AM ET | MEDIUM — leading CPI indicator |
| GDP | Quarterly, 8:30 AM ET | HIGH — recession/growth signal |
| Jobless Claims | Every Thursday, 8:30 AM ET | MEDIUM — labor market health |
| Retail Sales | ~15th of month, 8:30 AM ET | MEDIUM — consumer spending |
| FOMC Decision | 8x/year, 2:00 PM ET | EXTREME — rate changes move everything |
| PCE (Fed's preferred inflation) | Last Friday monthly, 8:30 AM ET | HIGH — Fed watches this most |
| ISM Manufacturing | First business day monthly | MEDIUM — economic expansion/contraction |

**Where to check:** `WebSearch "economic calendar today"` or [Investing.com Economic Calendar](https://www.investing.com/economic-calendar/)

### 8. Overnight News Sweep (5:00 AM PT)
Run these WebSearch queries every single session:

```
WebSearch "stock market news today"
WebSearch "market moving news overnight"
WebSearch "Fed news today"
WebSearch "geopolitical news markets"
WebSearch "crypto news today"
```

---

## How to Scan (ORACLE's Tool Stack)

### WebSearch Queries (run every session)
```
1. WebSearch "pre-market movers today"
2. WebSearch "stock futures today"
3. WebSearch "bitcoin price today"
4. WebSearch "crypto news today"
5. WebSearch "economic calendar today"
6. WebSearch "earnings reports today"
7. WebSearch "stock market news today"
8. WebSearch "VIX today"
```

### Alpha Vantage MCP
Pull real-time quotes on watchlist stocks. Prioritize:
- Core positions (any stock you hold)
- Sector ETFs (SPY, QQQ, XLK, XLE, XLF, etc.)
- Watchlist targets (stocks near entry/exit levels)

### Glassnode MCP (for crypto)
Check overnight on-chain data:
- BTC exchange netflow (7-day MA — ignore single-day spikes)
- Stablecoin exchange reserve (dry powder)
- Miner outflow (selling pressure)
- Long-term holder supply change
- Funding rates on perpetual futures

---

## Action Framework

| Pre-Market Signal | Bias | Action |
|-------------------|------|--------|
| Futures up 1%+ | BULL | Favor long setups, calls, bull spreads. Avoid opening shorts. |
| Futures down 1%+ | BEAR | Favor put spreads, inverse ETFs. Tighten stops on longs. |
| Futures flat (+/- 0.3%) | NEUTRAL | Range-bound day likely. Sell premium (iron condors, strangles). |
| Stock gaps up 5%+ on earnings beat | BULL (stock) | Watch for opening range breakout after first 15 min. Don't chase the gap. |
| Stock gaps down 5%+ on earnings miss | BEAR (stock) | Watch for gap fill setup (bullish) or continuation short. Wait for confirmation. |
| Stock gaps up 5%+ with NO news | INVESTIGATE | Could be M&A leak, FDA, or insider activity. Research before trading. |
| BTC up 5%+ overnight | CRYPTO BULL | Check funding rates — if negative, more room to run. If extreme positive, reversal risk. |
| BTC down 5%+ overnight | CRYPTO BEAR | Check exchange inflows. If spiking, more selling ahead. If flat, could be a dip to buy. |
| VIX above 25 | FEAR | Elevated premium — sell options strategies. Wider stops. Reduce size. |
| VIX below 15 | COMPLACENT | Cheap protection — buy puts/hedges. Volatility expansion coming. |
| VIX 15-25 | NORMAL | Standard playbook. No adjustment needed. |
| Economic data beats big | SHORT-TERM BULL | Trade the reaction in first 30 min. Indices pop, then often fade. |
| Economic data misses big | SHORT-TERM BEAR | Immediate sell-off. Watch for oversold bounce for entry. |
| Hot CPI / hawkish Fed | BEAR | Growth stocks sell off. Favor value, energy, financials. |
| Cool CPI / dovish Fed | BULL | Growth stocks rip. Favor tech, discretionary, small caps. |

---

## Pre-Market Watchlist Template

Fill this out every morning before market open. No exceptions.

```
=== ORACLE PRE-MARKET SCAN ===
Date: ____
Time completed: ____

--- MARKET DIRECTION ---
/ES (S&P futures): ____ (___%)
/NQ (Nasdaq futures): ____ (___%)
/YM (Dow futures): ____ (___%)
/RTY (Russell futures): ____ (___%)

--- VOLATILITY ---
VIX: ____
VIX trend (rising/falling/flat): ____

--- CRYPTO ---
BTC: $____ (___% 24h)
ETH: $____ (___% 24h)
SOL: $____ (___% 24h)
BTC Dominance: ____%
Crypto Fear & Greed: ____ (____)

--- INTERNATIONAL ---
Nikkei: ____%
Shanghai: ____%
FTSE: ____%
DAX: ____%

--- PRE-MARKET MOVERS ---
Top gapper up: ____ (___%) — Catalyst: ____
Top gapper down: ____ (___%) — Catalyst: ____
Unusual volume: ____ — Volume vs avg: ____x

--- EARNINGS TODAY ---
Pre-market: ____
After-hours: ____
Key reports to watch: ____

--- ECONOMIC DATA ---
Time: ____ | Release: ____ | Expected: ____ | Prior: ____

--- OVERNIGHT NEWS ---
Headline 1: ____
Headline 2: ____
Headline 3: ____

--- MY ASSESSMENT ---
Overall bias: BULL / BEAR / NEUTRAL
Confidence (1-10): ____
Reason: ____

--- TOP 3 SETUPS ---
1. ____ — Entry: ____ | Target: ____ | Stop: ____ | R:R: ____
2. ____ — Entry: ____ | Target: ____ | Stop: ____ | R:R: ____
3. ____ — Entry: ____ | Target: ____ | Stop: ____ | R:R: ____

--- RISK CHECK ---
Max position size today: $____
Max daily loss limit: $____
Open positions at risk: ____
Hedges in place: ____
```

---

## Pre-Market Scan Workflow (Timed)

| Time (PT) | Action | Duration |
|-----------|--------|----------|
| 5:00 AM | Check futures, VIX, overnight crypto, Asia close | 10 min |
| 5:10 AM | WebSearch news sweep (8 queries above) | 10 min |
| 5:20 AM | Check pre-market movers, volume leaders | 10 min |
| 5:30 AM | Economic data check, earnings calendar | 10 min |
| 5:40 AM | Glassnode on-chain check (crypto days) | 10 min |
| 5:50 AM | Alpha Vantage watchlist quotes | 10 min |
| 6:00 AM | Fill out watchlist template | 10 min |
| 6:10 AM | Identify top 3 setups, set alerts | 10 min |
| 6:20 AM | Final bias assessment, position sizing | 10 min |
| 6:30 AM | READY. Market opens at 6:30 AM PT. |  |

---

## Key Rules

1. **Never trade the first 5 minutes.** Let the opening chaos resolve. The opening range (first 15 min) gives you direction.
2. **Gaps fill 70% of the time.** A stock gapping up on weak volume often fills back. A stock gapping up on massive volume on a real catalyst often keeps running.
3. **The pre-market scan is NOT a trade plan.** It's information gathering. The trade plan comes from your [[pre-trade-checklist]] and [[options-setups-playbook]].
4. **If you skip the scan, you don't trade.** Period. No scan = no edge = gambling.
5. **Log every scan.** Even when nothing is interesting. The discipline of scanning IS the edge.

---

## Related ORACLE Files
- [[pre-trade-checklist]]
- [[options-setups-playbook]]
- [[risk-dashboard]]
- [[sector-rotation-model]]
- [[earnings-whisper-system]]
- [[market-regime-detector]]

---

*Sources: [CNBC Markets](https://www.cnbc.com/markets/), [Benzinga Pre-Market](https://www.benzinga.com/premarket), [StockAnalysis](https://stockanalysis.com/markets/premarket/), [MarketChameleon](https://marketchameleon.com/Reports/PremarketTrading), [TradingView](https://www.tradingview.com/markets/stocks-usa/market-movers-pre-market-gainers/), [Blockchain Magazine Crypto Market](https://blockchainmagazine.net/crypto-market-today-2026-04-23/), [Investing.com](https://www.investing.com/equities/pre-market)*
