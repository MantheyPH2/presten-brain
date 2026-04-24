---
type: trading-tool
name: Active Scanner System
last_updated: 2026-04-23
scan_frequency: every 15 minutes during market hours
---

# ORACLE Active Scanner System

Run this system every 15-minute session during market hours (6:30 AM - 1:00 PM PT for equities, 24/7 for crypto). Each scanner has specific search queries, filtering criteria, and action triggers.

---

## 1. Momentum Scanner

### What to Find
Stocks and crypto showing unusual strength with volume confirmation. These are names breaking out RIGHT NOW.

### Search Criteria
| Filter | Threshold | Why |
|--------|-----------|-----|
| Price change today | >+3% | Minimum move to qualify as momentum |
| Volume vs 20d avg | >2x average | Volume confirms the move is real, not noise |
| New 52-week high | Yes, with volume >1.5x | Breakout with conviction |
| Breaking resistance | Price through key level identified in tracker files | Technical breakout confirmed |

### How to Scan (WebSearch Queries)
```
"stocks up most today volume" site:finviz.com
"top gainers today premarket" site:benzinga.com
"crypto top gainers 24h" site:coingecko.com
"52 week high stocks today" site:stockanalysis.com
"unusual volume stocks today" site:marketchameleon.com
```

### MCP Tools to Use
- **WebSearch:** Run queries above every session for real-time data
- **WebFetch:** Pull finviz.com/screener.ashx for custom screener results
- **Read:** Check tracker files in `/Market/stocks/` and `/Market/tokens/` for key levels being tested

### Filtering Rules
1. **PASS** only if the stock/crypto is in our coverage universe (28 stocks, 40 tokens) OR has market cap >$1B
2. **REJECT** low-float penny stocks, micro-cap crypto below $50M market cap
3. **REJECT** stocks moving on dilution (secondary offerings) or reverse splits
4. **PRIORITIZE** names already on the HOT or WARM watchlist (see tiered-watchlist.md)

### Action on Signal
- **Strong momentum (>5% with >3x volume):** Add to HOT watchlist immediately. Open the tracker file and update key levels. Identify entry point (pullback to VWAP or breakout retest)
- **Moderate momentum (3-5% with 2x volume):** Add to WARM watchlist. Set price alert at resistance level
- **52-week high breakout:** This is the highest-probability momentum setup. If it is in our universe, prepare a trade plan with entry, stop, and target

---

## 2. Mean Reversion Scanner

### What to Find
Oversold stocks and crypto that have been punished beyond what fundamentals justify. These are "blood in the streets" opportunities.

### Search Criteria
| Filter | Threshold | Why |
|--------|-----------|-----|
| RSI (14-day) | <30 | Classic oversold signal |
| VIX level | >25 | Fear index elevated = market-wide oversold |
| Price drop today | >-5% on no news | Overreaction without fundamental catalyst |
| Distance from 200-day MA | >-15% below | Stretched rubber band — mean reversion likely |
| Put/Call ratio | >1.2 | Excessive bearishness = contrarian buy signal |

### How to Scan (WebSearch Queries)
```
"most oversold stocks RSI today" site:finviz.com
"VIX index today" site:cboe.com
"stocks down most today no news" site:seekingalpha.com
"oversold crypto RSI" site:tradingview.com
"put call ratio today" site:cboe.com
"fear greed index" site:cnn.com
```

### MCP Tools to Use
- **WebSearch:** Run oversold queries when VIX >20 or market down >1%
- **WebFetch:** Pull CNN Fear & Greed Index for sentiment reading
- **Read:** Cross-reference drops against tracker files — is the stock dropping to a key support level?

### Filtering Rules
1. **PASS** only if the drop is NOT caused by: fraud, SEC investigation, bankruptcy, or fundamental business destruction
2. **PASS** only if the company/protocol is profitable or has >12 months cash runway
3. **REJECT** "falling knife" stocks in confirmed downtrends (lower highs and lower lows for >3 months)
4. **PRIORITIZE** quality names (SPY components, top-50 crypto by market cap) that are oversold

### Action on Signal
- **RSI <25 on a quality name:** Add to HOT watchlist. This is a high-probability mean reversion. Set buy order at support level from tracker file
- **VIX >30:** Market-wide fear event. Run the full portfolio stress test. Look for quality names at 200-day MA support. Prepare "crash buy" list
- **Stock down >5% on no news in our universe:** Investigate immediately. If no fundamental cause found, this is likely an algo-driven or options-driven overreaction. Buy the dip with tight stop at next support level
- **VIX >40:** Extreme fear. Deploy the "crash buy" playbook from the COLD watchlist. Maximum aggression on quality names

---

## 3. Options Scanner

### What to Find
Options pricing anomalies that create edge. High IV = sell premium. Low IV = buy options. Unusual volume = follow smart money.

### Search Criteria
| Filter | Threshold | Why |
|--------|-----------|-----|
| IV Rank | >60 | Expensive options — sell premium (credit spreads, iron condors) |
| IV Rank | <20 | Cheap options — buy calls/puts for directional bets |
| Options volume vs OI | >3x on specific strikes | Unusual activity — potential smart money positioning |
| Put/Call skew | Extreme (>1.5 or <0.5) | Directional sentiment in options market |
| Earnings in <7 days | IV rank >70 | Sell premium into earnings IV crush |

### How to Scan (WebSearch Queries)
```
"unusual options activity today" site:marketchameleon.com
"highest IV rank stocks today" site:barchart.com
"options unusual volume today" site:optionistics.com
"earnings options IV crush" site:marketchameleon.com
"dark pool activity today" site:flowalgo.com
"options sweep alerts" site:benzinga.com
```

### MCP Tools to Use
- **WebSearch:** Run unusual options queries every session during market hours
- **WebFetch:** Pull barchart.com options screener for IV rank data
- **Read:** Check tracker files for upcoming earnings dates and key levels

### Filtering Rules
1. **PASS** only if the stock is in our coverage universe or has options with decent liquidity (bid-ask spread <$0.20)
2. **REJECT** illiquid options (volume <100 contracts, wide spreads)
3. **REJECT** meme-driven options spikes (WSB-driven pump with no fundamental thesis)
4. **PRIORITIZE** unusual volume on blue-chip names — that is more likely institutional positioning

### Action on Signal
- **IV Rank >60 with earnings in <7 days:** Prepare iron condor or credit spread. Sell premium into earnings IV crush. Use the options-setups-playbook.md for specific setup
- **IV Rank <20 on a stock with upcoming catalyst:** Buy calls/puts for directional leverage. Cheap options + catalyst = asymmetric payoff
- **Unusual volume >5x on specific strikes:** Investigate. If it aligns with our thesis, follow the smart money. If it contradicts, reassess
- **Dark pool prints >$1M:** Institutional positioning. Note the direction and add to watchlist

---

## 4. Crypto Scanner

### What to Find
On-chain and derivatives signals unique to crypto markets. Funding rates, exchange flows, and whale movements.

### Search Criteria
| Filter | Threshold | Why |
|--------|-----------|-----|
| Funding rate | >0.05% or <-0.05% | Extreme positioning — mean reversion likely |
| Exchange inflow spike | >2 standard deviations | Potential sell pressure (moving to exchange to sell) |
| Exchange outflow spike | >2 standard deviations | Accumulation (moving off exchange to hold) |
| Whale wallet movements | >$10M in single transaction | Institutional or whale positioning |
| Open Interest change | >10% in 24h | Significant new positions being opened |
| Liquidation cascade | >$100M in 1h | Forced selling/buying creating opportunity |

### How to Scan (WebSearch Queries)
```
"crypto funding rates" site:coinglass.com
"bitcoin exchange inflow outflow" site:cryptoquant.com
"whale alert crypto today" site:whale-alert.io
"crypto liquidations today" site:coinglass.com
"bitcoin open interest" site:coinglass.com
"crypto fear greed index" site:alternative.me
"Glassnode on-chain analysis" site:glassnode.com
```

### MCP Tools to Use
- **WebSearch:** Run crypto-specific queries every session (crypto is 24/7)
- **WebFetch:** Pull coinglass.com for funding rates and liquidation data
- **Read:** Cross-reference with token tracker files for context

### Filtering Rules
1. **PASS** only for tokens in our 40-token coverage universe
2. **REJECT** signals on tokens below $100M market cap (too illiquid for reliable signals)
3. **REJECT** whale movements that are internal exchange transfers (not real accumulation/distribution)
4. **PRIORITIZE** signals on BTC and ETH — they drive the entire market

### Action on Signal
- **Funding rate >0.05% (extreme positive):** Longs are overleveraged. Expect a pullback. Consider closing long positions or opening small shorts. Do NOT fight this signal
- **Funding rate <-0.05% (extreme negative):** Shorts are overleveraged. Expect a squeeze (like the April 23 BTC squeeze). This is the highest-probability crypto signal. BUY into extreme negative funding
- **Exchange outflow spike on BTC/ETH:** Bullish — large holders moving to cold storage = accumulation. Confirms bullish thesis
- **Exchange inflow spike on BTC/ETH:** Bearish — potential sell pressure incoming. Tighten stops on crypto positions
- **>$200M liquidations in 1h:** Forced selling is done. The bottom is likely in. Start scaling into positions
- **Whale wallets accumulating below key support:** Follow the whales. If they are buying the dip, you should be too

---

## Scanner Execution Schedule

### During US Market Hours (6:30 AM - 1:00 PM PT)
| Time | Scanner | Priority |
|------|---------|----------|
| 5:00-6:30 AM | Pre-market momentum + crypto overnight | HIGH |
| 6:30-7:00 AM | Opening drive momentum | HIGHEST |
| 7:00 AM | Mean reversion (check RSI levels) | HIGH |
| 9:00 AM | Options scanner (mid-day volume) | MEDIUM |
| 10:30 AM | Power hour setup scan | HIGH |
| 12:30 PM | End-of-day positioning | MEDIUM |

### After Hours / Overnight
| Time | Scanner | Priority |
|------|---------|----------|
| 1:00-1:30 PM | After-hours earnings reactions | HIGH |
| 5:00 PM | Crypto scanner (evening check) | MEDIUM |
| 10:00 PM | Crypto overnight setup | LOW |

### Weekend
| Time | Scanner | Priority |
|------|---------|----------|
| Saturday morning | Crypto scanner only | LOW |
| Sunday evening | Pre-week crypto positioning scan | MEDIUM |

---

## Scanner Output Format

Every scanner hit gets logged in this format:

```
### [TIMESTAMP] — [SCANNER TYPE] — [TICKER]
- **Signal:** [What triggered]
- **Price:** [Current price]
- **Volume:** [vs average]
- **Key Level:** [Nearest support/resistance from tracker]
- **Action:** [Add to watchlist / Enter position / Set alert / Monitor]
- **Conviction:** [1-5 scale]
- **Notes:** [Context]
```

Log scanner hits in the daily trading journal entry.
