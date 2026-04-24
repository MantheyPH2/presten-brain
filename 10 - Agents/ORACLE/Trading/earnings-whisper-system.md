# ORACLE Earnings Whisper System

> **Use before EVERY earnings trade | The whisper number is the real target, not consensus**
> A company can "beat earnings" and the stock dumps. Understanding WHY is the difference between making money and losing it on earnings.

---

## What Are Whisper Numbers

### The Problem with Consensus
- **Consensus estimate** = the average of all published Wall Street analyst EPS forecasts
- This number is PUBLIC. Everyone knows it. It's priced in.
- Companies have been "beating consensus" 70-75% of the time for over a decade
- If beating consensus were the signal, stocks would go up 75% of the time after earnings. They don't.

### The Whisper Number
- **Whisper number** = what traders ACTUALLY expect the company to report
- It's the unpublished, unofficial expectation — often higher than consensus
- Analysts update their models with new data but don't always publish revised estimates. Their internal number (the whisper) is more current.
- The stock moves based on beat/miss vs the WHISPER, not consensus

### The Proof
According to EarningsWhispers.com data (25+ years, 117,000+ whisper numbers):
- The whisper number has been **closer to actual EPS than consensus 70% of the time**
- Stocks that beat the whisper closed higher 60% of the time, averaging +1.8% gains
- Stocks that beat consensus BUT missed the whisper **closed LOWER 55% of the time**
- This is the single most important insight for earnings trading: beating consensus means nothing if you miss the whisper

---

## Where to Find Whisper Numbers

### Primary Sources

| Source | URL | What It Offers | Cost |
|--------|-----|---------------|------|
| **EarningsWhispers.com** | earningswhispers.com | Proprietary whisper numbers, earnings calendar, whisper grades | Free (basic) / Premium |
| **TheWhisperNumber.com** | thewhispernumber.com | Consensus vs whisper comparison, 2,300+ stocks | Free (basic) / Premium |
| **Estimize** | estimize.com | Crowdsourced estimates from 100,000+ contributors (buy-side, sell-side, students, independent) | Free (basic) |

### Secondary / Derived Sources

| Source | What It Tells You | How to Use It |
|--------|-------------------|-------------|
| **Options implied move** | What the market is ACTUALLY betting the stock will move | This IS the market's whisper — it prices in all expectations |
| **Unusual options activity** | Large call buying = someone expects upside surprise. Large put buying = downside | Pre-earnings unusual activity is a whisper signal |
| **Social media sentiment** | Reddit, X, StockTwits consensus before earnings | Crowdsourced whisper — especially useful for retail-heavy stocks |
| **Insider buying/selling** | Insiders buying before earnings = confidence. Selling = concern | SEC Form 4 filings — check before every trade |
| **Guidance revisions** | Did the company pre-announce or update guidance? | A pre-announcement usually sets a new (higher) whisper |

### How ORACLE Sources Whisper Data
Run these WebSearch queries before every earnings trade:
```
WebSearch "[TICKER] earnings whisper number"
WebSearch "[TICKER] earnings estimates Estimize"
WebSearch "[TICKER] options implied move earnings"
WebSearch "[TICKER] unusual options activity"
WebSearch "[TICKER] insider buying selling"
```

---

## How to Calculate Implied Move from Options

The options market prices in a specific expected move around earnings. This is the most reliable "whisper" because it represents REAL MONEY being bet.

### The Straddle Method (Simple)

**Step 1:** Find the nearest-expiration at-the-money (ATM) options (the expiration immediately after earnings).

**Step 2:** Add the ATM call price + ATM put price = straddle price.

**Step 3:** Multiply by 0.85 (the "85% rule" — accounts for time value decay).

**Step 4:** Divide by the stock price = implied percentage move.

### Worked Example

```
NVDA reports earnings. Stock is at $150.
ATM Call (150 strike, nearest expiry): $8.50
ATM Put (150 strike, nearest expiry): $7.50
Straddle price: $8.50 + $7.50 = $16.00
Adjusted: $16.00 x 0.85 = $13.60
Implied move: $13.60 / $150 = 9.07%

Interpretation: The options market expects NVDA to move
+/- 9% after earnings. This is a 1 standard deviation move
(68% probability the stock stays within this range).

Expected range: $136.40 to $163.60
```

### What the Implied Move Tells You

| Implied Move vs Historical | Meaning | Strategy |
|---------------------------|---------|----------|
| Implied move > historical average move | Options are EXPENSIVE. Market expects a bigger move than usual. | Sell premium — iron condors, strangles. Collect inflated premium. |
| Implied move < historical average move | Options are CHEAP. Market is underpricing the move. | Buy premium — straddles, strangles. You're getting a deal. |
| Implied move = historical average move | Fairly priced. No edge in direction-agnostic strategies. | Only trade if you have directional conviction. |

### Where to Find Historical Implied Moves
- `WebSearch "[TICKER] historical earnings move vs implied"`
- MarketChameleon earnings analysis
- OptionStrat earnings calendar
- ThinkOrSwim earnings tab

---

## Earnings Whisper Playbook

### The Four Earnings Outcomes

| Scenario | What Happens | Stock Reaction | Probability | Strategy |
|----------|-------------|---------------|------------|----------|
| **Beat consensus AND whisper** | Genuine positive surprise. Stock rips. | +5% to +20% gap up. Often continues for 2-3 days. | ~25% | Buy calls or bull call spreads before earnings. Hold through report. |
| **Beat consensus, MISS whisper** | "Beat" is a mirage. Traders expected more. Stock dumps despite headline "beat." | -3% to -10% despite positive headline. Confuses retail. | ~30% | This is the trap. Sell premium. Iron condors. Or put spreads if you see the whisper gap. |
| **Miss consensus AND whisper** | Genuine negative surprise. Stock dumps. | -10% to -25%. Analysts downgrade. Selling continues 1-3 days. | ~20% | Put spreads before. Or wait for the gap down and trade the dead cat bounce / mean reversion 2-5 days later. |
| **Miss consensus, BEAT whisper** | Looks bad on headlines but traders expected worse. Stock recovers. | Initial -3% to -8% gap down, then recovery within 1-3 days. | ~25% | Buy the dip after the initial panic sell. Stock often fills the gap within a week. |

### Strategy Matrix by IV Rank

| IV Rank | What It Means | Earnings Strategy |
|---------|--------------|-------------------|
| **IV Rank > 70** | Options are very expensive relative to their 52-week range | SELL premium. Iron condors, short strangles, short straddles. The inflated IV will crush after earnings (IV crush). |
| **IV Rank 40-70** | Options are moderately priced | Directional spreads if you have conviction. Bull/bear put/call spreads. |
| **IV Rank < 40** | Options are cheap relative to their range | BUY premium if you expect a big move. Long straddles, long strangles. Cheap entry. |

**Where to find IV Rank:** `WebSearch "[TICKER] IV rank"` or check on barchart.com, marketchameleon.com, or your broker's platform.

---

## Pre-Earnings Checklist

Before EVERY earnings trade, answer all 10 questions. If you can't answer them all, you don't have enough information to trade.

### The 10 Questions

**1. What is the consensus EPS?**
- Source: Yahoo Finance, Seeking Alpha, or `WebSearch "[TICKER] earnings estimate"`
- Write it down: Consensus EPS = $____

**2. What is the whisper number?**
- Source: EarningsWhispers.com, TheWhisperNumber.com, Estimize
- Write it down: Whisper EPS = $____
- Gap: Whisper is ____% above/below consensus

**3. What is the options implied move?**
- Calculate using straddle method above
- Write it down: Implied move = +/- ____%
- Expected range: $____ to $____

**4. Is the implied move higher or lower than the historical average move?**
- `WebSearch "[TICKER] historical earnings move"`
- Average historical move: +/- ____%
- Current implied move vs historical: HIGHER / LOWER / INLINE
- Interpretation: Options are EXPENSIVE / CHEAP / FAIR

**5. What is the IV rank?**
- Current IV rank: ____%
- Strategy implication: SELL premium / DIRECTIONAL spread / BUY premium

**6. What is the KEY METRIC to watch (not just EPS)?**
Every company has a metric that matters MORE than EPS:

| Company | Key Metric Beyond EPS |
|---------|---------------------|
| AAPL | iPhone units, Services revenue, China sales |
| AMZN | AWS revenue growth, operating margins, ad revenue |
| GOOGL | Search ad revenue, Cloud revenue, YouTube |
| META | Daily Active Users, ad revenue per user, Reality Labs losses |
| MSFT | Azure growth rate, Copilot adoption, Office 365 subscribers |
| NVDA | Data center revenue, gross margins, guidance |
| TSLA | Deliveries, automotive margins, energy storage |
| NFLX | Subscriber adds, ARM (ad revenue per member), churn |
| JPM | Net interest income, credit losses, trading revenue |
| DIS | Disney+ subscribers, Parks revenue, streaming profitability |

Write it down: Key metric = ____, expected = ____, whisper = ____

**7. What did management guide last quarter?**
- Previous quarter guidance: Revenue $____ | EPS $____
- Is the company tracking above or below their own guide?
- Guidance matters as much as the current quarter's results

**8. What are insiders doing?**
- `WebSearch "[TICKER] insider buying selling SEC filings"`
- Check SEC Form 4 filings at sec.gov
- Net insider activity (90 days): BUYING / SELLING / NEUTRAL
- Large insider purchases before earnings = very bullish signal
- Insider selling can be routine (options exercises) — look for unusual size

**9. What did sector peers report?**
- If MSFT beat on cloud, GOOGL likely beats on cloud too
- If one chip company missed, check if the weakness is sector-wide
- Peer results: ____ (beat/miss) → Implication for [TICKER]: ____

**10. What is my specific trade structure?**
- Direction: BULLISH / BEARISH / NEUTRAL
- Structure: ____ (calls, puts, spread, condor, straddle)
- Entry price: $____
- Max risk: $____
- Max reward: $____
- Risk/reward ratio: ____
- Exit plan if right: ____
- Exit plan if wrong: ____

---

## Post-Earnings Playbook

### The First 15 Minutes After Report
- DO NOT trade in the first 5 minutes. Spreads are wide. Price is chaotic.
- Wait for the earnings call to start (usually 30-60 min after report)
- The call often matters more than the numbers — guidance and tone drive the next move

### Reading the Reaction

| Reaction | What It Means | Next Move |
|----------|--------------|-----------|
| Stock gaps up and HOLDS above gap | Strong beat + strong guidance. Institutions buying. | Hold longs. Add on pullback to gap level. |
| Stock gaps up but FADES during the day | "Sell the news." Beat was priced in. | Take profits. This often continues lower for 2-3 days. |
| Stock gaps down and CONTINUES lower | Bad report + bad guidance. Analysts will downgrade. | Don't catch the knife. Wait 2-3 days for stabilization. |
| Stock gaps down but REVERSES higher | Initial panic, then buyers step in. Whisper beat despite headline miss. | Buy the reversal. Very bullish signal. |
| Stock barely moves despite big beat/miss | Implied move was too high. IV crush kills options. | If you sold premium, collect your profit. |

### Earnings Drift
Research shows that stocks continue to drift in the direction of the earnings surprise for 60-90 days after the report. This is called "Post-Earnings Announcement Drift" (PEAD).

- Stocks that beat estimates tend to continue rising for weeks
- Stocks that miss tend to continue falling for weeks
- This is one of the most persistent anomalies in financial markets
- Trade it: Buy the strongest earnings beats 2-3 days after the report (after the initial volatility settles)

---

## Earnings Season Calendar

Major earnings seasons and what to expect:

| Period | Reporting Quarter | Key Dates | What to Watch |
|--------|-----------------|-----------|---------------|
| Jan 15 - Feb 15 | Q4 results | Banks report first week. Tech last 2 weeks. | Full-year guidance for new year |
| Apr 15 - May 15 | Q1 results | Same pattern. Watch for guidance revisions. | Consumer spending, seasonal trends |
| Jul 15 - Aug 15 | Q2 results | Mid-year check. Back-to-school previews. | Margin pressure, input costs |
| Oct 15 - Nov 15 | Q3 results | Holiday guidance is critical. | Holiday outlook, inventory levels |

**Current context (April 2026):** Q1 2026 earnings season is underway. Intel beat massively (+19% after-hours). Tesla results came in mixed with the stock slipping. Watch for mega-cap tech reports in the next 2 weeks.

---

## Earnings Whisper Score Template

Fill this out for every earnings trade:

```
=== ORACLE EARNINGS WHISPER ANALYSIS ===
Date: ____
Ticker: ____
Report date/time: ____

--- ESTIMATES ---
Consensus EPS: $____
Whisper EPS: $____
Whisper gap: ___% above/below consensus
Consensus Revenue: $____B
Whisper Revenue: $____B

--- OPTIONS DATA ---
ATM Call price: $____
ATM Put price: $____
Straddle price: $____
Implied move: +/- ____%
Historical avg move: +/- ____%
Implied vs historical: EXPENSIVE / CHEAP / FAIR
IV Rank: ____%

--- KEY METRIC ---
Metric: ____
Consensus: ____
Whisper: ____
Previous quarter actual: ____

--- CONTEXT ---
Management guidance (last Q): ____
Insider activity (90d): BUYING / SELLING / NEUTRAL
Peer results: ____
Sector trend: ____

--- TRADE PLAN ---
Direction: BULLISH / BEARISH / NEUTRAL
Confidence (1-10): ____
Structure: ____
Entry: $____
Max risk: $____
Target: $____
R:R ratio: ____

--- POST-EARNINGS RESULT ---
Actual EPS: $____
Beat/miss consensus by: ____%
Beat/miss whisper by: ____%
Key metric actual: ____
Stock move: ____%
Implied move captured: YES / NO (___%)
Trade P&L: $____ (___%)
Lesson learned: ____
```

---

## Key Rules for Earnings Trading

1. **The whisper is the bar.** Beating consensus means nothing if the whisper is higher. Always check.
2. **IV crush is real.** Options lose 30-60% of their value the day after earnings regardless of direction (implied volatility collapses). If you're buying options, you need a BIG move to profit.
3. **Sell premium when IV rank is above 50.** This is the highest-probability strategy in earnings trading. The house always wins when options are expensive.
4. **Never risk more than 2% of portfolio on a single earnings trade.** Binary events are gambling. Size accordingly.
5. **The earnings call matters more than the numbers.** Listen for guidance, tone, and management confidence. The stock often reverses direction during the call.
6. **Trade the drift, not the event.** The safest earnings trade is buying the strongest beats 2-3 days after the report and riding the drift for 2-4 weeks.
7. **Sector peers are your crystal ball.** If 3 of 5 companies in a sector beat, the remaining 2 probably will too.
8. **Insiders know more than you.** If insiders are buying before earnings, pay attention. They're not gambling.
9. **The options market is usually right about the SIZE of the move.** The implied move is accurate to within 1-2% roughly 70% of the time. The market is wrong about direction more often.
10. **Keep a record of every earnings trade.** Track your accuracy on whisper predictions. Improve over time.

---

## Related ORACLE Files
- [[pre-trade-checklist]]
- [[options-setups-playbook]]
- [[pre-market-scanner]]
- [[risk-dashboard]]
- [[trading-journal]]

---

*Sources: [Earnings Whispers](https://www.earningswhispers.com/about-whispers), [The Whisper Number](https://thewhispernumber.com/), [Wikipedia - Whisper Number](https://en.wikipedia.org/wiki/Whisper_number), [Motley Fool - Whisper Number Definition](https://www.fool.com/terms/w/whisper-number/), [Power Trading Group - Options Implied Move](https://www.powertrading.group/options-trading-blog/options-implied-move), [Options Hawk - Expected Moves](https://optionshawk.com/calculating-expected-moves-using-options/), [MenthorQ - Straddle to Expected Move](https://menthorq.com/guide/from-straddle-price-to-expected-move/), [Bullish Bears - Earnings Whispers Review 2026](https://bullishbears.com/earnings-whispers/)*
