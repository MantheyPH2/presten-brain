---
tags: [oracle, trading, sentiment, news, fundamentals]
created: 2026-04-21
---

# News Sentiment Scoring

## The Problem with Subjective Sentiment

"Bullish news" is not a tradeable signal. "Very bullish" vs. "slightly bullish" vs. "priced in" are three entirely different situations that look identical on the surface. Every trader has read a headline, felt optimistic, entered a trade — and watched the stock drop. The news was good. The sentiment was already baked in.

ORACLE needs a systematized, repeatable scoring framework that:
1. Forces explicit evaluation rather than gut reaction
2. Accounts for magnitude, scope, duration, and surprise
3. Produces a number that maps directly to a position action
4. Creates an audit trail — so you can see whether your scoring was accurate

---

## Sentiment Score: -10 to +10

| Score | Label | Example |
|-------|-------|---------|
| **+10** | Paradigm shift bullish | Fed cuts 100bps unexpectedly; war ends; transformative regulation passes (crypto ETF approval); breakthrough cure discovered |
| **+8** | Major catalyst bullish | Earnings blowout + guidance raise 20%+; major acquisition at premium; activist investor takes large stake with improvement plan |
| **+6** | Strong bullish | Earnings beat + guidance raise; competitor goes bankrupt; major contract win; index inclusion |
| **+5** | Solid bullish | Earnings beat (EPS and revenue), in-line guidance raise; positive FDA ruling; credit upgrade |
| **+3** | Mild bullish | Earnings meet expectations with no downside surprise; analyst upgrade with new target; positive sector data |
| **+1** | Neutral-positive | Normal session, no news, market drifting up; minor positive rumor unconfirmed |
| **0** | True neutral | Nothing happening; no catalyst; consolidation day |
| **-1** | Neutral-negative | Minor concern surfacing; vague rumor; insider selling reported |
| **-3** | Mild bearish | Earnings miss on EPS or revenue (not both); analyst downgrade; sector headwind; minor guidance cut |
| **-5** | Solid bearish | Clear bad news — layoffs announced; guidance cut 10%+; product recall; CEO departure unexpectedly |
| **-7** | Major catalyst bearish | Earnings disaster (miss + significant guidance cut); major regulatory action; criminal investigation announced; major customer lost |
| **-8** | Severe bearish | Earnings disaster with going-concern warning; SEC investigation; accounting irregularities discovered; major lawsuit with large potential damages |
| **-10** | Paradigm shift bearish | Black swan event; systemic financial crisis; war escalation; exchange hack/collapse; government ban on core business |

---

## The Five-Factor Scoring Method

Do not assign a score based on headline alone. Score each of the five factors, then synthesize:

### Factor 1: Impact Scope
- **Single stock only:** Max ±4 on the scale (affects only this position)
- **Sector-wide:** Max ±6 (affects all names in the sector)
- **Market-wide / macro:** Full ±10 (affects all risk assets)

*Why it matters:* A biotech's FDA approval is a +8 for that stock but a +1 for the market. A Fed rate cut decision is a ±8 for the whole market. Conflating these leads to over-positioning.

### Factor 2: Duration of Effect
- **One-day event (binary, then forgotten):** Reduces score weight by ~20%
- **Week-long positioning shift:** Normal weight
- **Structural/lasting change (months to years):** Increases score weight by ~20%

*Examples:* An earnings beat is a one-time event — the sentiment boost fades in days. A regulatory change that permanently expands a company's addressable market is structural — it deserves full weight and then some.

### Factor 3: Surprise Factor
- **Fully expected by market (already priced in):** Reduce score by 2–3 points. "Buy the rumor, sell the news" — the market already moved.
- **Partially expected:** Normal score
- **Complete surprise:** Increase score by 2–3 points. The market has to reprice from scratch — the move will be bigger and more sustained.

*Calibration tool:* Check options pricing before the event. If IV was elevated, the market expected a move. If IV was flat, the market was surprised. A stock with low IV before a massive earnings beat = maximum surprise = maximum sustained move.

### Factor 4: Magnitude
- **Small beat, small miss:** ±1–3 range
- **Significant beat or miss (EPS 10%+ above/below expectations):** ±5–7 range
- **Total blowout or disaster (25%+ from expectations):** ±8–10 range

*Numbers matter:* An earnings beat of $0.01 vs. $0.00 expected is not the same as beating by $0.50 vs. $0.40 expected. Scale your reaction to the size of the deviation.

### Factor 5: Historical Comparison
Before assigning a final score, **find a similar past event.** What happened?
- "Company X beat by 15% and raised guidance in 2022 → stock up 18% in 3 days"
- "Competitor received same FDA rejection in 2021 → sector sold off 5% over 2 weeks"
- "Central bank surprise cut in 2019 → market rallied 3% on day 1, faded over next week"

Historical analogs prevent recency bias and anchoring to the current day's emotional reaction.

---

## Scoring to Action: The Decision Matrix

```
SENTIMENT SCORE → POSITION ACTION

+9 to +10:  AGGRESSIVE LONG
            Size up to maximum allowed position
            Add on any dip the first 3 days
            Stop: Below the catalyst day's low

+7 to +8:   STRONG LONG
            Full position size
            Enter on first pullback or open if volume confirms
            Stop: Below catalyst day low, or -5% from entry

+4 to +6:   BULLISH
            Normal position size (no oversizing)
            Standard entry with standard stop
            Take partial profits at +10–15%

+1 to +3:   SLIGHT BULL LEAN
            Half position or wait for technical confirmation
            No urgency — let the setup develop
            Sentiment alone is not enough

-3 to -1:   SLIGHT BEAR LEAN
            Tighten stops on existing longs
            No new long entries on affected names
            Watch for deterioration

-6 to -4:   BEARISH
            Reduce longs 25–50% in affected names/sector
            Add hedges (puts, inverse ETFs, reduce delta)
            Consider modest short on technically weak setup

-8 to -7:   STRONGLY BEARISH
            Exit most or all affected longs
            Initiate short positions if structure confirms
            No buying the dip until dust settles (minimum 48 hours)

-10 to -9:  CRASH PLAYBOOK
            See risk-dashboard.md
            Exit longs, activate hedges, raise maximum cash
            Do not try to catch falling knives
```

---

## Adjustments for "Priced In" vs. "Not Priced In"

The single most common mistake in news trading: **reacting to the news rather than to the market's reaction to the news.**

**Signs a positive event was already priced in:**
- Stock had already rallied 20-30% in the weeks leading up to the catalyst
- Options IV was elevated before the announcement (market expected it)
- Stock barely moves or drops on "good news" — that is the signal
- Volume is below average on the positive catalyst

**Adjustment:** Reduce score by 2–3 points. A +8 event becomes a +5 effective signal if it was largely anticipated.

**Signs a positive event was NOT priced in:**
- Stock had been flat or declining before the catalyst
- IV was low (nobody was hedging against a big move)
- Stock gapped up strongly on massive, above-average volume
- Analyst community had not upgraded the name recently

**Adjustment:** Increase score by 2–3 points. The repricing process will take longer and go further.

---

## Sector Contagion Rules

When a major event hits one company, it often affects the whole sector:
- **Competitor bankruptcy/major loss:** Score +4 to +6 for surviving competitors (less competition, market share gains likely)
- **Regulatory crackdown on one name:** Score -4 to -6 for entire sector (regulation often expands to sector)
- **Strong earnings from sector bellwether:** Score +2 to +3 for sector peers (rising tide)
- **Weak earnings from sector bellwether with systemic warning:** Score -3 to -5 for sector peers

---

## The Macro Override Rule

If the market-wide sentiment is -7 or worse (crisis, systemic risk), **individual stock sentiment is largely irrelevant.** During 2008, 2020, and 2022 drawdowns, stocks with +8 sentiment still fell 30–50% because all correlations went to 1.0 and everything sold off together.

When market-wide sentiment is -7 or below:
- Reduce all individual stock scores by 3–5 points
- No new longs regardless of individual catalyst
- Focus on cash preservation and hedging

---

## Daily Sentiment Dashboard

Complete every morning before market open:

```
ORACLE DAILY SENTIMENT DASHBOARD
==================================
Date: ____

MACRO INPUTS
-----------
Fed stance: HAWKISH / NEUTRAL / DOVISH
Recent data: GDP ____ | CPI ____ | NFP ____
VIX level: ____ (below 15 = calm | 15-25 = normal | 25-35 = elevated | 35+ = fear)
10yr yield: ____ (direction: RISING / FALLING / FLAT)

NEWS SCORING
-----------
Top event #1: ______________________ Score: __/10
Top event #2: ______________________ Score: __/10
Top event #3: ______________________ Score: __/10

SENTIMENT TOTALS
-----------
Market sentiment (aggregate): ____/10
Crypto sentiment:             ____/10
Top bullish catalyst today:   ______________________________
Top bearish risk today:       ______________________________

POSITIONING LEAN
-----------
Overall: AGGRESSIVE BULL / BULL / NEUTRAL / BEAR / AGGRESSIVE BEAR
Cash target: ____%
Hedge level: NONE / LIGHT (5-10%) / MODERATE (10-20%) / HEAVY (20%+)
```

---

## Calibration Log

Track your scores vs. outcomes to improve accuracy:

```
SENTIMENT CALIBRATION LOG
==========================
Date | Event | Score Assigned | Actual Outcome | Error Notes
-----|-------|---------------|---------------|------------
     |       |               |               |
```

Review monthly. If you consistently over-score positive events (assigned +7, outcome was +3), reduce your positive bias by 1–2 points going forward. Good scoring systems improve through honest feedback loops.
