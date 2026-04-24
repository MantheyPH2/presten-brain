---
title: DSS Demo Spec — May 2026
aliases: [DSS Demo, DSS Rankings Demo]
tags: [dss, demo, rankings, product, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: final
dss_date: 2026-05-16
demo_ready_by: 2026-05-09
---

# DSS Demo Spec — Rankings Showcase, May 16, 2026

> **Core claim: Evo Draw ranks teams more accurately than USA Rank — and covers more of them.**
>
> Every element of this demo serves that claim. Nothing in this doc is a "maybe." Each step names a specific team, event, or club. Presten reads this on May 14 and knows exactly what to show.

---

## Section 1: Core Claim and 60-Second Pitch

### The Claim

> "USA Rank is the incumbent. We're more accurate, and we cover 40% more teams. Here's the proof."

### The 60-Second Verbal Pitch

> "USA Rank has been the standard for youth soccer rankings for years. They rank the teams everyone's heard of. We agree with them on the elite teams — and then we go deeper. We cover USYS state leagues, GA, regional cups. Teams that USA Rank has no data on, we've ranked. We use Glicko-2, which is the same system that powers competitive chess and esports — it accounts for opponent quality, not just wins. And unlike USA Rank, which resets every 20 games and uses goal margin, our system rewards consistent long-term excellence. A team that wins ugly against strong opponents gets full credit. Today I want to show you three things: rank bands, event strength cards, and club rankings. None of them exist on USA Rank the way we've built them."

### The Anchor Number

**"We rank teams with full game history — not just last 20 games."**

This is the one-sentence differentiation that resonates with coaches. A parent sees their team's ranking and wonders "why?" We can show them the trajectory: every game, every opponent, every rating move. USA Rank cannot.

---

## Section 2: Primary Demo Flow

Execute these 5 steps in order. Each step has a named team/event/club — do not improvise on demo day.

---

### Step 1: Rankings Page — U13 Girls, Top 10 with Rank Bands

**URL:** `/rankings?age_group=U13&gender=F`

**What to show:**
- Top 10 rankings for U13G with Gold/Silver band badges visible
- Specifically call out that The Football Academy NJ 2012 Girls Black appears near the top

**The claim:**
> "USA Rank has Football Academy NJ at #4. We have them at [Evo Draw rank]. We agree on who's elite. And we have teams ranked below #100 that USA Rank has no data on at all — because we crawl data sources they don't."

**Required data state by May 9:**
- `rankings_with_bands` view deployed
- U13G has 100+ active ranked teams
- Football Academy NJ visible by name in top 50

**What to point to:**
- The Gold badge next to Football Academy NJ: "That badge means they're in the top tier — same concept as USA Rank's Gold, but our thresholds are grounded in actual Elo distribution, not rank count."

---

### Step 2: Team Detail Page — Full Game History

**URL:** `/teams/[Football Academy NJ team_id]` (confirm ID before May 14 by running Query 1A from the Accuracy Audit)

**What to show:**
- Game history list: all games, opponent names, scores, rating changes
- Rating trajectory chart if available (rating over time)
- Total game count vs. the "last 20 games" USA Rank limitation

**The claim:**
> "USA Rank shows you the last 20 games. We show you every game this team has ever played in our system — [X] games going back to [earliest date]. A team that's been consistently excellent for 2 years gets full credit. A team on a 3-week hot streak gets appropriate weight, not a free pass to the top."

**Required data state by May 9:**
- Team detail page exists and shows game history
- Football Academy NJ has 30+ games in our system

---

### Step 3: Event Page — High-Strength Event with Strength Card

**Event to feature:** The highest-strength event in U13G or U14G in the 2025-2026 season that has a recognizable name to a NJ/PA/NE audience. Target: an ECNL Regional or GA National event.

**Find this event before May 14:**
```sql
SELECT event_name, age_group, gender, team_count,
       avg_team_rating, strength_label, strength_percentile, rating_spread, upset_rate
FROM event_strength
WHERE season = '2025-2026'
  AND gender = 'F'
  AND age_group IN ('U13','U14')
  AND strength_label = 'High'
ORDER BY avg_team_rating DESC
LIMIT 10;
```

Pick the event with the most recognizable name to the DSS audience (northeast corridor coaches and club directors).

**URL:** `/events/[event name encoded]?age_group=U13G`

**What to show:**
- Event Strength Card: "High" badge + avg team rating (e.g., 1,321) + percentile (e.g., 91st) + Tight/Mixed spread + upset rate %
- Contrast it with USA Rank: "They show you one label — 'High.' We show you this event ranked in the 91st percentile of all U13G events, the field averaged 1,321 Elo, and 23% of decisive games were upsets."

**The claim:**
> "USA Rank shows you a label. We show you the math. Coaches can decide for themselves whether a 'High' event with a Wide spread is actually what they want their team in."

**Required data state by May 9:**
- `event_strength` table populated (Phase 1+2 of Event Strength implementation complete)
- Target event has `strength_label = 'High'` and numeric values computed
- Event detail page shows `EventStrengthCard` component

---

### Step 4: Club Rankings Page — NJ/PA/NE Club

**URL:** `/club-rankings?gender=F`

**Club to feature:** A well-known NJ or PA girls club. Candidates (confirm before May 14):
1. **PA Classics** — PA-based ECNL powerhouse, strong U13–U17 girls program
2. **PDA (Players Development Academy)** — NJ-based, top girls club in the region
3. **NJ Wildcats** — NJ-based, strong multi-age-group program

To confirm which club to use, run before May 14:
```sql
SELECT c.name, cr.club_rank, cr.club_rating, cr.age_groups_counted
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE cr.gender = 'F'
  AND (c.name ILIKE '%PA Classics%' OR c.name ILIKE '%PDA%' OR c.name ILIKE '%Wildcats%'
       OR c.name ILIKE '%NJ%' AND cr.club_rank <= 50)
ORDER BY cr.club_rank
LIMIT 10;
```

Pick the highest-ranked NJ/PA club that the audience is likely to recognize by name.

**What to show:**
- Club rankings table (girls): PA Classics at rank X, rating Y, 7/7 age groups
- Click into club profile: show age group breakdown table (U11 through U17, each team name + rating)

**The claim:**
> "A club director walks in here and in 10 seconds can see where their program ranks nationally, broken down by every age group. USA Rank has club rankings too — but they require 5 of the specific age groups they cover. We cover 7 age groups and flag exactly which ones you have teams in."

**Required data state by May 9:**
- `clubs` and `club_rankings` tables seeded and populated
- PA Classics (or chosen club) has 5+ active age groups in rankings
- `/club-rankings` page renders
- `/clubs/:id` profile shows age group breakdown

---

### Step 5: Coverage Comparison — "We Go Deeper"

**This is the closing statement, not a feature click.**

**The number to cite:**
- "We have [X] ranked teams in U13G. USA Rank has approximately [Y]." (Fill in X from a count query before May 14; Y is not publicly known but the point is directional — we rank more teams from state leagues.)
- "5 of the 16 northeast teams USA Rank has publicly ranked are not in our system yet — we know exactly which data sources we need to add. That's the roadmap. For the teams we do have, our rankings match USA Rank's ordering."

```sql
-- Run before May 14 — get our U13G ranked team count
SELECT COUNT(*) AS ranked_u13g_teams
FROM rankings
WHERE age_group = 'U13' AND gender = 'F'
  AND last_game_date > NOW() - INTERVAL '7 months';
```

**The claim:**
> "We're not claiming we have everything USA Rank has. We're claiming we have everything they have on ECNL, GA, and MLS NEXT — and we have state league data they don't. The accuracy audit we ran shows our top-50 rankings closely match theirs for the teams we share. The gap is source coverage, not methodology."

---

## Section 3: Specific Teams, Events, and Clubs

### Teams for U13G Comparison

| Role | Team | USA Rank | Expected Evo Draw | 1-Sentence Explanation |
|------|------|----------|--------------------|------------------------|
| Validates methodology | The Football Academy NJ 2012 Girls Black | #4 | Top 20 expected | ECNL team with dense GotSport data; Glicko and USA Rank should converge for elite teams |
| Methodology divergence | NE Surf G2012 State Navy | #198 | Possibly not found OR different rank | "State Navy" = state association team; if lower in Evo Draw, it's source coverage — we don't have NEYSA state cup data yet |
| Coverage advantage | SYC 2012G GA II | #826 | Check vs audit (GA calibration test) | GA calibration validation confirms or adjusts the 140-point GA bonus; if we rank them higher, explain the empirical backing |

**Before May 14:** Run the accuracy audit Query 1A to fill in the Evo Draw Rank column for Football Academy NJ. If they appear in top 20, this is the headline comparison. If they appear outside top 50, investigate the merge (FA 12G Black and Football Academy NJ may be split records — check Query 1B).

### Events for Event Strength Demo

| Role | How to Find | Requirements |
|------|-------------|-------------|
| Primary (High strength) | Top result from the SQL query in Section 2 Step 3 | Recognizable tournament name, High label, U13G or U14G |
| Contrast (Average/Low) | Second result from same query, pick one with Average label | Same age group for apples-to-apples comparison |

**Before May 14:** Run the event strength query and lock in the specific event name. If the event name is a generic internal label (e.g., "Event 12345"), do not use it — pick the next one with a human-readable name.

### Club for Club Rankings Demo

**Primary choice:** PA Classics (if ranked in top 50 girls nationally with 5+ age groups)
**Backup:** PDA (NJ), then NJ Wildcats

**Before May 14:** Run the club confirmation query from Section 2 Step 4. Lock in the club name and record its rank and rating.

---

## Section 4: Data Readiness Checklist

Every item must be green by **May 9** (demo-ready deadline).

| Feature | Requirement | Owner | Risk | Status |
|---------|-------------|-------|------|--------|
| Rank Bands | `rankings_with_bands` view exists; top U13G teams have correct band badges | Presten (~5.5h session) | **Low** — plan written, no blockers | Pending implementation |
| Event Strength | `event_strength` table populated for 2025-2026 season; target event identified | Presten (Phase 1+2, ~7h; must start by April 28) | **Medium** — must start immediately | Pending implementation |
| Club Rankings | `clubs` seeded; `club_rankings` computed; PA Classics (or backup) identified | Presten (~10h session; start by May 2) | **Medium** — plan written today | Pending implementation |
| USA Rank comparison | Football Academy NJ Evo Draw rank confirmed via SQL; FA 12G merge check | FORGE or Presten (30-min DB task) | **Low** — SQL is written in Accuracy Audit | Pending DB run |
| Source coverage number | U13G ranked team count for the "we go deeper" claim | Presten (5-minute query) | **Low** — one query | Pending |

### Red flags to check May 9:

1. **If Event Strength is not ready:** Fall back to the Section 5 demo (Rank Bands only). Do not show an empty event page.
2. **If Club Rankings is not ready:** Skip Step 4 entirely. The demo is still strong with Steps 1–3.
3. **If Football Academy NJ is not in our system:** Use a different top-10 team from the Accuracy Audit Query 1D (full U13G rankings). The point is "we agree on top-10," not specifically this team.
4. **If GA calibration audit found issues:** Do NOT cite the SYC 2012G GA case as a positive example. The GA calibration validation is a separate workstream — omit it from the demo if the result is uncertain.

---

## Section 5: Fallback Demo (Rank Bands Only)

If only Rank Bands ships by May 9, this is the complete demo. It is still compelling.

### Fallback: 3 clicks, 5 minutes, one strong claim

**Step 1:** Rankings page → U13G → Gold badge next to Football Academy NJ
> "Same team USA Rank has at #4. We have them at [rank]. We agree on who's elite."

**Step 2:** Team detail page → show game history
> "USA Rank uses last 20 games. We use every game. This team has [X] games. Consistent excellence over 18 months gets full credit."

**Step 3:** Show the band filter — select "Gold" → only top teams remain
> "Coaches can filter by tier. Instead of scrolling 500 teams, you see the Gold-tier teams in your age group. This is the feature USA Rank has. We ship it with better thresholds — ours are based on actual Elo distribution, not arbitrary rank cutoffs."

**Closing:**
> "The event strength cards and club rankings are coming. The accuracy and coverage are already there. What you're seeing today is the engine that powers this — the rest is UI."

The fallback demo makes the same core claim with less surface area. It is not a weaker demo — it is a focused one.

---

## Section 6: What NOT to Demo

These are explicit exclusions. Do not show these, do not reference them, do not answer questions about them without flagging uncertainty.

1. **U12 or U19 rankings specifically.** The calibration changes for these age groups have not been applied yet. While U19 is only mildly miscalibrated (Brier 0.238 vs 0.23 threshold), a knowledgeable observer asking about a specific U19 team's ranking could expose a known gap. Stick to U13–U17 where calibration is confirmed sound or under active remediation.

2. **Any specific team rank that depends on a pending FORGE SQL result.** The comparison table in the Accuracy Audit has `[QUERY]` placeholders. Do not quote Evo Draw ranks for Football Academy NJ or Maryland Rush unless FORGE has run the queries and confirmed the numbers before May 14.

3. **Calibration parameter changes.** "We recently discovered our U13 model was overconfident and fixed it" is a story with two possible audiences: one that appreciates rigor and one that hears "your rankings were wrong." Do not introduce this at DSS. The fix will be applied post-DSS (May 18-20). By DSS the rankings shown use the current (pre-fix) calibration.

4. **State league source gaps by name.** Saying "we don't have NEYSA data" in front of coaches whose teams play in NEYSA is not a selling point. The framing is "we're adding sources" not "we're missing sources." If asked directly, acknowledge the roadmap without naming specific gaps.

5. **Any feature flagged as "pending Presten approval."** This includes calibration changes. These are internal workstream details.

---

## Pre-Demo Day Checklist (May 14)

Run through this 48 hours before DSS:

- [ ] `rankings_with_bands` view confirmed working — `SELECT COUNT(*) FROM rankings_with_bands WHERE rank_band = 'Gold' AND age_group = 'U13' AND gender = 'F'` returns > 0
- [ ] Football Academy NJ confirmed in Evo Draw with rank < 50 for U13G (or backup team identified)
- [ ] Target High-strength event locked in by name (not just query result)
- [ ] PA Classics (or backup club) confirmed in `club_rankings` with rank and rating recorded
- [ ] U13G active team count recorded for "we go deeper" talking point
- [ ] All 5 demo URLs tested in a browser with real data
- [ ] Fallback path confirmed — if Event Strength or Club Rankings is not ready, know which steps to skip

---

## Related Notes

- [[USA Rank Competitive Intel]] — competitor features, bands, event pages, club rankings
- [[USA Rank Ranking Accuracy Audit — April 2026]] — methodology comparison and SQL queries
- [[Rank Bands — Implementation Plan]] — Rank Bands feature (Step 1, Step 2 of demo)
- [[Event Strength Ratings — Implementation Plan]] — Event Strength feature (Step 3 of demo)
- [[Club Rankings — Implementation Plan]] — Club Rankings feature (Step 4 of demo)
- [[Calibration Validation 2026-04-22]] — calibration state (do not mention at DSS)
