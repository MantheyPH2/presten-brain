---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-04-27
---

# Task: Write DSS Demo Spec — Ranking Showcase for May 16

## Context

DSS is May 16. There are 23 days to get data clean, features built, and a demo path ready. Right now, we have three approved implementation plans (Rank Bands, Event Strength Ratings, Club Rankings) and four pending FORGE tasks. The problem: no one has defined what the demo actually looks like.

This task: write the DSS Demo Spec. A concrete, opinionated document that answers — for the rankings/data story — what Presten shows, which teams, which events, which features, and what narrative ties it together. This is ELO's call because ELO understands the data quality, the calibration state, and which comparisons hold up.

The demo must make one strong claim: **Evo Draw ranks teams more accurately than USA Rank.** Every element of the spec should serve that claim.

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — their product, their bands, their event pages, their club rankings
- `02 - Tiger Tournaments/Projects/Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — your methodology divergence analysis
- `02 - Tiger Tournaments/Projects/Rankings/Calibration Validation 2026-04-22.md` — current state of calibration quality
- `02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Implementation Plan.md` — Rank Bands feature spec
- `02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings — Implementation Plan.md` — Event Strength feature spec
- `02 - Tiger Tournaments/Projects/Rankings/Club Rankings Design.md` — Club Rankings feature spec

## What to Produce

### Section 1: Core Demo Claim and Narrative

Write the 60-second verbal pitch for the rankings demo at DSS. It should answer:
- What does Evo Draw do that USA Rank doesn't?
- Why does that matter to the coaches and club directors in the room?
- What is the one number or comparison that lands the claim?

The pitch must be grounded in data we actually have (or will have by May 9). Identify which data point is the anchor — e.g., "We rank 40% more teams than USA Rank in U13G because we capture USYS state league data they miss." Use the accuracy audit and GA coverage findings to support claims.

### Section 2: Primary Demo Flow (Recommended)

Specify the exact sequence of what Presten clicks/shows. Each step: what page, what feature, what claim.

**Suggested flow — adapt based on what will be built by May 9:**

1. **Rankings Page — U13 Girls** → Show top 10 with Rank Bands (Gold/Silver badges visible). Compare top 3 vs USA Rank's top 3. "We agree on the elite tier — and we go deeper."

2. **Click into a top-10 team** → Team detail page. Show game history, events, Glicko rating trajectory. "We use full game history — not just last 20 games. Teams with consistent long-term performance are rewarded."

3. **Event Page — High-Strength Event** → Show Event Strength card: High, avg rating 1420, Tight spread, 35% upset rate. Compare: "USA Rank shows one label. We show you the math." Pick a real tournament known to the audience (GA or ECNL regional if covered).

4. **Club Rankings Page** → Show top clubs for U13G. "A club director can immediately see their club's standing across all age groups." Scroll to a NJ/NE club the audience recognizes.

5. **Coverage comparison** → "USA Rank covers X leagues. We cover Y, including USYS state leagues they miss." Use the USYS State Cup research finding (20,000–35,000 additional games via org ID expansion, if completed by then).

For each step: specify which specific team/event/club to use (don't leave this to chance on demo day).

### Section 3: Specific Teams and Events to Feature

Based on the accuracy audit and coverage analysis, specify:

**3 teams to anchor the U13G comparison:**
- 1 team where our rank closely matches USA Rank's (validates methodology)
- 1 team where our rank is meaningfully different (with a defensible explanation — more data, better calibration)
- 1 team USA Rank has ranked that we rank higher (shows our deeper coverage advantages certain teams)

For each: USA Rank position, our expected position (or position once FORGE runs the SQL), and the 1-sentence explanation for any divergence.

**2 events to feature for Event Strength:**
- 1 High-strength event (major ECNL or GA regional) — should have recognizable name
- 1 mixed-strength event — for contrast

**1 club to feature for Club Rankings:**
- A well-known NJ/PA/NE club (this is the DSS audience's geography)
- Should have teams in at least 6 age groups

Document the specific names so FORGE can verify the data is clean before May 16.

### Section 4: Data Readiness Checklist

For each feature in the demo, what needs to be true in the DB by May 9:

| Feature | Requirement | Owner | Risk |
|---------|-------------|-------|------|
| Rank Bands | `rankings_with_bands` view exists, top teams have correct bands | Presten (5.5h session) | Low — plan ready |
| Event Strength | `event_strength` table populated for 2025–2026 events | Presten (Phase 1+2, 7h) | Medium — must start by Apr 28 |
| Club Rankings | `clubs` and `club_rankings` tables seeded and computed | Presten (8–12h session) | Medium — plan pending |
| USYS org expansion | At least TX org ID added, test crawl completed | FORGE | Medium — task assigned Apr 24 |
| USA Rank comparison | Comparison table populated with Evo Draw data | FORGE (running SQL Apr 24) | Low — 30-minute task |

Flag any item with risk = High and propose a fallback if it's not complete by May 9.

### Section 5: Fallback Demo (If Only Rank Bands Ships)

If only Rank Bands is implemented by May 9, what does the demo look like? Write the shortened version that still makes the core claim with just:
- Rankings page with band badges
- Team detail page
- The USA Rank comparison table (accuracy audit results)

The fallback must still be compelling. Do not design a demo that requires all three features to land.

### Section 6: What NOT to Demo

Explicitly list what to avoid showing:
- Any age group where calibration validation is incomplete or known to be off (U12? U19?)
- Any team or region where game coverage is known to be thin
- Any feature that depends on pending Presten approval (calibration changes must not be referenced before approval)
- Any claim that depends on FORGE's April 24 SQL results until those results are reviewed

This section protects against overreach. The best demo is a narrow, tight story — not a feature tour.

## What Done Looks Like

Produce the spec at:
`02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo Spec — May 2026.md`

The spec must be:
- Opinionated — not "here are options," but "here is what to show and why"
- Data-anchored — every claim references a specific known number or feature state
- Actionable — someone reading it on May 14 knows exactly what to demo

## Success Criteria

- Primary demo flow is specified with named teams, events, and clubs (not "a team" — a specific team)
- Data readiness checklist is complete and has a named owner for each item
- Fallback demo is specified and still compelling with only Rank Bands shipped
- Section 6 (what NOT to demo) has at least 4 specific exclusions
- Core claim is stated and defensible with current data
