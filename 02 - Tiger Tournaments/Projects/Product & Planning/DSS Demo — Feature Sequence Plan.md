---
title: DSS Demo — Feature Sequence Plan
type: demo-plan
author: ELO
date: 2026-04-25
status: draft
version: draft-pre-may9
update_trigger: May 9 readiness assessment
related: "[[DSS Feature Readiness Tracker — May 2026]]", "[[DSS Demo Spec — May 2026]]", "[[DSS Ranking Accuracy Claims — Authorized Spec]]"
tags: [dss, demo, sequence, product, rankings, evo-draw]
---

# DSS Demo — Feature Sequence Plan

> **Draft — valid as of 2026-04-25. Update after May 9 readiness assessment. File final version as `DSS Demo — Feature Sequence Plan — Final.md` before May 12.**
>
> Items marked `[CONDITIONAL]` may be removed if the corresponding authorization gate does not clear by May 9.

---

## Section 1: Demo Parameters

```
Target audience: DSS stakeholders — assumed to include club directors and league administrators
  who know what USA Rank is and can evaluate rankings claims credibly
Duration: 15 minutes
Format: Live database + Evo Draw UI (psql as fallback for any feature not yet in UI)
Goal: After the demo, the audience should believe Evo Draw's rankings are more accurate
  than USA Rank for elite teams AND cover teams USA Rank misses entirely.
```

The goal shapes sequencing: open with the claim the audience can immediately verify (do we agree on top teams?), then add depth (full game history), then add features USA Rank doesn't have (event strength, club rankings), then close with coverage breadth. The arc is: credibility → depth → differentiation → breadth.

---

## Section 2: Feature Inventory

| Feature | Authorization Status | ELO Confidence | Demo Value (1–5) | Dependency |
|---------|---------------------|----------------|-----------------|-----------|
| Rank Bands | Conditional: pending April 28 recompute + G3 gate | High | **5** | April 28 execution; threshold re-validation post-fix |
| Team Detail / Game History | Authorized (no separate gate — present if rankings page works) | High | **4** | Rankings page exists |
| Event Strength Phase 1 | Conditional: pending April 29 G2 gate + 7h session | Medium | **4** | April 28 fix; Phase 1 session May 2–3 |
| Club Rankings (Girls) | Conditional: pending calibration stability + 10h session | Medium | **5** | April 29 calibration authorization; May 2–5 session |
| Club Rankings (Boys) `[CONDITIONAL]` | Conditional: pending Boys Option A verdict (May 5) | Medium | **4** | Branch A: Option A PASS |
| USA Rank Comparison | Conditional: pending FORGE SQL run | High | **3** | 30-min Presten psql session |
| Brier Score / Predictive Accuracy | Conditionally authorized (post-April-29 update) | Medium | **3** | April 29 analysis complete |
| Game Coverage Narrative | Authorization TBD | Low | **2** | TX org ID + test crawl |

**Top Demo Value features:** Club Rankings (5), Rank Bands (5), Event Strength Phase 1 (4), Team Detail (4).

**Demo Value guide used:** 5 = best standalone story for a non-technical audience with no further context. 1 = interesting only to a technical audience.

---

## Section 3: Proposed Sequence (Draft)

Assumes primary demo: all features authorized except USYS/GotSport Coverage. Runs approximately 15 minutes.

---

**1. Rank Bands — U13G Rankings Page — [2.5 min]**

What to show: `/rankings?age_group=U13&gender=F` — top 10 with Gold/Silver band badges. Football Academy NJ visible near top.

What to say: "USA Rank has Football Academy NJ at #4. We have them at [Evo Draw rank]. We agree on who's elite — and we have teams ranked below #100 that USA Rank has no data on at all, because we crawl sources they don't. That Gold badge means they're in the top tier. Our threshold is based on actual Elo distribution, not rank count."

Compelling comparison: Football Academy NJ at USA Rank #4 vs. Evo Draw rank (should be top 20). Agreement at the top is the credibility anchor.

Transition to next item: "Let me show you what's behind that ranking."

---

**2. Team Detail — Full Game History — [2.5 min]**

What to show: `/teams/[Football Academy NJ team_id]` — game history list with opponent names, scores, rating changes, and rating trajectory chart.

What to say: "USA Rank shows you the last 20 games. We show you every game this team has ever played in our system — [X] games going back to [earliest date]. A team that's been consistently excellent for two years gets full credit. A team on a 3-week hot streak gets appropriate weight, not a free pass to the top. Coaches can see exactly why a team is ranked where it is — every opponent, every result."

Compelling comparison: USA Rank's 20-game window vs. Evo Draw's full game history for the same team.

Transition to next item: "Now let me show you something USA Rank doesn't have at all — event context."

---

**3. Event Strength Phase 1 — High-Strength Event Card — [3 min]** `[CONDITIONAL — remove if Phase 1 session does not complete before May 9]`

What to show: Event detail page for the highest-strength U13G or U14G event in 2025–2026. Find via: `SELECT event_name, strength_class, strength_percentile, avg_team_rating, upset_rate FROM event_strength WHERE season='2025-2026' AND gender='F' AND age_group IN ('U13','U14') AND strength_class='High' ORDER BY avg_team_rating DESC LIMIT 10;` — pick the most recognizable event name.

What to say: "USA Rank labels events as 'High' or 'Low.' We show you the math behind the label. This event ranked in the [N]th percentile of all U13G events this season. The average team rating was [X]. [N]% of decisive games were upsets. A coach deciding whether to enter this tournament can see whether 'High strength' means elite competition or just a well-attended regional. That's information USA Rank doesn't surface."

Compelling comparison: USA Rank label vs. Evo Draw's percentile, avg_team_rating, upset_rate breakdown.

Transition to next item: "Now let me show you club rankings — an entirely new view of the data."

---

**4. Club Rankings — Girls Program View — [3 min]** `[CONDITIONAL — remove if Club Rankings session does not complete before May 9]`

What to show: `/club-rankings?gender=F` — top 20 clubs nationally. Click into PA Classics (or PDA or NJ Wildcats — whichever ranks highest with 5+ age groups). Show age group breakdown table.

What to say: "A club director walks in and in 10 seconds can see where their program ranks nationally — not just one team, every age group at once. PA Classics [or chosen club] is ranked [rank] nationally. You can see U11 through U17, each team's rating and rank. USA Rank has club rankings too, but they require 5 of the specific age groups they cover. We cover 7 age groups and every team we have data on is in here."

Compelling comparison: Evo Draw's multi-age-group club aggregate vs. USA Rank's club rankings (narrower coverage).

*If Boys Option A passes — add one sentence:* "And we have it for Boys too — [top Boys club name] is ranked [rank] nationally on the Boys side."

Transition to next item: "The last point is why this is more than a better USA Rank — it's broader."

---

**5. Coverage Comparison — Closing Statement — [2 min]**

What to show: Simple coverage count from `SELECT COUNT(*) FROM rankings WHERE age_group='U13' AND gender='F' AND last_game_date > NOW() - INTERVAL '7 months';`. No page navigation needed — state the number.

What to say: "We have [X] ranked U13G teams. USA Rank ranks approximately [Y] — their public lists show around 2,000 teams per age group nationally. Five of the 16 northeast teams USA Rank publicly ranks are not in our system yet — we know exactly which sources to add. That's the roadmap. For the teams we share, our rankings match their ordering. The accuracy is there. The coverage gap is a data sourcing problem, not a methodology problem — and we're closing it."

Compelling comparison: Our team count vs. USA Rank's approximate count; state the coverage gap candidly with a roadmap framing.

No transition — this is the close.

---

## Section 4: Contingency Sequences

**Fallback A — Boys Option A fails (Girls-only demo):**
Remove the Boys sentence from Step 4 entirely. The Club Rankings step shows Girls only. No other reordering needed — Boys was an addendum within Step 4, not a standalone step. The narrative arc (credibility → depth → differentiation → breadth) is unchanged. Opening and closing are Girls-focused from the start. Duration stays approximately 15 minutes.

**Fallback B — Club Rankings not ready (no May 9 authorization or implementation not complete):**
Remove Step 4 entirely. Step 3 (Event Strength) becomes the differentiation closer. Add 30 seconds to Steps 1–3 to use the freed time. The closing statement (Step 5) pivots: "Event strength cards and club rankings are coming — the rankings engine and accuracy are already there. What you're seeing today is the foundation." Duration shrinks to approximately 12 minutes; this is acceptable. The Fallback Demo from `DSS Demo Spec — May 2026` Section 5 covers the case where both Event Strength and Club Rankings are missing (Rank Bands only, ~5 minutes).

---

## Section 5: Specific Examples and Reference Teams/Clubs

| Entity | Why This One | Feature It Demonstrates | Age Group |
|--------|-------------|------------------------|-----------|
| Football Academy NJ 2012 Girls Black | USA Rank #4 — the highest-ranked NE team we expect in our system; ECNL team with dense GotSport data; Glicko and USA Rank should converge | Rank Bands (Step 1), Team Detail (Step 2) | U13G |
| PA Classics (primary) | PA-based ECNL powerhouse with strong U13–U17 girls program; well-known to NE corridor audience; strong candidate for Girls club top-20 | Club Rankings (Step 4) | Multi-age-group |
| PDA (backup) | NJ-based, top Girls club in the NJ/PA region; recognizable to DSS audience | Club Rankings (Step 4) | Multi-age-group |
| Highest-rated U13G or U14G ECNL/GA event in 2025-26 | Best anchor for "High-strength" claim; lock in specific event name before May 14 | Event Strength (Step 3) | U13G or U14G |
| SYC 2012G GA II | USA Rank #826 — GA-affiliated team; if Evo Draw ranks higher, use as calibration explanation; if lower, do not feature | USA Rank comparison context only | U13G |

**Boys conditional examples (use only if Branch A activated):**
- Highest-ranked Boys club by girls-comparable criteria — ELO identifies after Option A passes
- Omit entirely if Branch B

**Confirm before May 14:** Run Step 1A query from `USA Rank Comparison 2026-04-23.md` for Football Academy NJ. If they appear in Evo Draw top 20, this is the headline. If outside top 50, check team_merges for FA 12G Black / Football Academy NJ merge status before flagging a problem.

---

## Section 6: May 9 Update Protocol

After the May 9 readiness assessment (`May 9 DSS Readiness Final Assessment Spec.md`):
1. Update Section 2 Feature Inventory with final authorization statuses (✅ / ❌)
2. Update Section 3 sequence: remove any `[CONDITIONAL]` items that did not clear; remove the Boys sentence from Step 4 if Option A fails
3. Update Section 5 with final reference teams (run confirmation queries for Football Academy NJ and chosen club)
4. File updated version as `DSS Demo — Feature Sequence Plan — Final.md` in this folder
5. Note in briefing: "Final demo sequence plan filed. Features in sequence: [N]. Duration estimate: [N] minutes. Contingency notes: [any]."

---

## References

- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — feature authorization status
- `Product & Planning/DSS Demo Spec — May 2026.md` — overall demo objectives and flow
- `Rankings/Club Rankings — Reference Clubs.md` — reference clubs for Section 5
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — authorized accuracy claims for talking points
- `Rankings/Boys Option A — Verdict Filing Document.md` — determines Boys features in Section 4 Fallback A
- `Rankings/May 9 DSS Readiness Final Assessment Spec.md` — triggers the May 9 update to this document

*ELO — 2026-04-25 (draft)*
