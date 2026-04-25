---
title: Event Strength Phase 1 — Rankings Impact Preview
type: prediction-document
author: ELO
date: 2026-04-25
status: pre-run preview
tags: [event-strength, prediction, phase-1, rankings, dss, evo-draw]
related: "[[Event Strength Phase 1 — Full Execution Package]]", "[[Event Strength Ratings — Implementation Plan]]", "[[Post-Fix Calibration Stability Monitoring Spec]]"
---

# Event Strength Phase 1 — Rankings Impact Preview

> **[Pre-Run Preview — Written before Event Strength Phase 1 execution]**
>
> This document is ELO's prediction of what Event Strength Phase 1 will and will not do to team rankings. Written April 25, 2026. Compare against post-Phase-1 analysis.

---

## Section 1: Mechanism Summary

**What Phase 1 actually does:** Event Strength Phase 1 creates a new `event_strength` table. It reads team ratings from the existing `rankings` table, computes four metrics per event — avg_team_rating, competitive_spread (σ), upset_rate, and strength_percentile — and stores them in `event_strength`. The computation uses current team ratings as a proxy for team quality at game time.

**What Phase 1 does NOT do:** Phase 1 does **not** modify the `rankings` table. It does **not** feed event strength back into the Glicko-2 computation. It does **not** change any team's rating. The `rankings` table is read-only input for Phase 1; the `event_strength` table is Phase 1's only output.

**Phase scope:** Phase 1 is a display and metadata feature. Event strength labels and metrics appear on event detail pages as a competitive differentiator (we show numbers, USA Rank shows only labels). The event strength modifier as a rating computation input is Phase 2 scope — not Phase 1.

**Does Phase 1 apply to all events or specific tiers?** Phase 1 covers all events in the `events` table with `event_tier IS NOT NULL` and team_count ≥ 4. No tier exclusion. The strength classification (High/Medium/Low) is percentile-based within (age_group, season) — so "High" means the top 25% of events by avg_team_rating for that age group, not a fixed absolute threshold.

If Phase 1 is run while the Section 1 remediation SQL is still pending (null event_tier events), those events receive `event_tier = 'local'` before computation and are included in the percentile calculation. This is intended — unclassified events should participate in the percentile distribution.

---

## Section 2: Affected Age Groups and Leagues

Because Phase 1 does not modify team ratings, the concept of "impact on ratings" by age group is zero for all age groups. However, **coverage** — how many events get strength ratings per age group — varies substantially. The table below estimates Phase 1 coverage, not rating impact.

| Age Group | Events Expected (2025-2026) | High-Strength Events Expected | Notes |
|-----------|-----------------------------|-------------------------------|-------|
| U13G | High (~150+) | ~38 (top 25%) | DSS demo cohort; ECNL + GA events well-represented |
| U13B | High (~130+) | ~33 | MLS NEXT Boys events included |
| U14G | High (~140+) | ~35 | Peak GA enrollment — high event volume |
| U14B | High (~120+) | ~30 | |
| U15G | Moderate (~100+) | ~25 | |
| U15B | Moderate (~90+) | ~23 | Peak MLS NEXT Boys years — high avg ratings in top events |
| U16G | Moderate (~90+) | ~23 | |
| U16B | Moderate (~80+) | ~20 | |
| U17G | Moderate (~80+) | ~20 | |
| U17B | Moderate (~70+) | ~18 | |

**Impact level definition (on event_strength coverage, not on rankings):**
- **High coverage:** > 100 events with strength ratings for that age group
- **Moderate coverage:** 50–100 events
- **Low coverage:** < 50 events (may indicate low game volume or thin event_tier tagging)

**Rating impact level for all age groups: ZERO.** Phase 1 does not modify team ratings. If any rating change is observed after Phase 1 runs, it is coincidental (e.g., the nightly pipeline ran between Phase 1 and the pre/post comparison) and not caused by Phase 1.

---

## Section 3: Predicted Rating Direction

Because Phase 1 does not modify ratings, ELO's predictions per age group are uniform:

**For all age groups:** Net rating change = 0 pts. Ratings do not increase or decrease as a result of running Phase 1.

**Rating spread:** Unchanged. Phase 1 does not widen or narrow any rating distribution.

**Redistribution:** None. Phase 1 produces event metadata only.

**What Phase 1 does change:** The ability to *display* and *filter* events by strength. Teams at top-tier events will be identifiable as having played in "High" events. This changes user perception of rankings context, but the numeric ratings are unchanged.

---

## Section 4: Anomaly Detection Thresholds

Because Phase 1 does not modify ratings, the anomaly signals are coverage-based:

| Signal | Expected | If Outside Range: Interpretation |
|--------|----------|----------------------------------|
| `event_strength` row count for 2025-2026 | ≥ 500 events | If < 100 → computation failed or event_tier remediation incomplete; re-run Gate B and Section 1 |
| High-strength events for U13G | ≥ 5 events | If 0 → possible age_group column format mismatch ('U13G' vs 'U13'/'F'); check schema |
| Any rating change in `rankings` table after Phase 1 | 0 pts | If any rating changed → nightly pipeline ran concurrently with Phase 1; compare timestamps |
| Strength class distribution | High ~25%, Medium ~42%, Low ~33% | If > 50% High → PERCENT_RANK partition may be missing season filter; check V2 validation query |
| Rows with null strength_class | 0 | If > 0 → CASE threshold logic has a gap; check the classified CTE in the computation SQL |

**Boys/Girls rating assertion after Phase 1:** Boys avg rating change must be 0 pts. Girls avg rating change must be 0 pts. Phase 1 does not touch ratings. Any observed rating change is from a concurrent pipeline run — check `computed_at` timestamps in rankings vs event_strength.

---

## Section 5: DSS Demo Implication

After Phase 1 runs as predicted, the following changes to the rankings story at DSS:

**Event pages gain strength cards.** The V4 validation query (`event_strength` lookup for High-strength U13G events) must return at least one recognizable High-strength event for U13G. This is the only Phase 1 deliverable that directly affects the DSS demo.

**Rankings story itself is unchanged.** Phase 1 does not validate, improve, or modify Evo Draw's team ranking accuracy. The accuracy claim ("we rank more teams than USA Rank, with more data") is unchanged by Phase 1.

**Competitive differentiator unlocked.** Phase 1 enables: "For the ECNL Showcase in the demo, we can show the audience that this event was High-strength, with an avg team rating of 1,421, 28% upset rate, Tight spread — here's the math behind that label." USA Rank shows a label only. This is the story Phase 1 enables.

**Should SENTINEL update the DSS demo script to reference event strength?** Yes — specifically, SENTINEL should confirm the named High-strength U13G event from the V4 output (run after Phase 1 completes) and reference it by name in the demo script (Step 3 of the DSS Demo Spec). The event name cannot be confirmed until Phase 1 runs and V4 is executed.

**Phase 2 context:** If SENTINEL is asked "does this affect team ratings?" at DSS, the honest answer post-Phase-1 is: no, event strength is a display and context feature today. It is the groundwork for a future improvement where highly competitive events may receive additional weight in rating computation. Do not claim Phase 2 is live at DSS unless Phase 2 has actually run.

---

## ELO Confidence Statement

Confidence in the zero-rating-impact prediction: **Definitive.** The Phase 1 SQL does not write to the `rankings` table. The prediction that ratings will not change is not probabilistic — it is an architectural certainty given the current Phase 1 implementation.

Confidence in coverage predictions (event counts): **Medium.** Estimates are based on the known event volume from the GA Coverage Audit and Cross-League Analysis game volumes. Actual event count depends on how many events survive the `HAVING COUNT(DISTINCT team_id) >= 4` minimum threshold. If large events have poor team-to-rankings join coverage (teams without rankings records), effective team_count may be below 4 for some events despite having more participating teams in the raw data.

If Phase 1 returns fewer than 100 events for 2025-2026, ELO investigates the join coverage issue before flagging Phase 1 as failing.

---

## References

- [[Event Strength Phase 1 — Full Execution Package]] — the implementation this preview models; Section 3 (computation INSERT) is the authoritative Phase 1 scope
- [[Event Strength Ratings — Implementation Plan]] — Phase 1 and Phase 2 scope definitions
- [[Event Strength Phase 2 — Scope and Plan]] — Phase 2 context: this is where event strength becomes a rating modifier
- [[Post-April-28 Expected Rating Shift — Pre-Run Model]] — template format
- [[Post-Fix Calibration Stability Monitoring Spec]] — May 5 stability gate that precedes Phase 1 authorization
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Phase 1 readiness cell

*ELO — 2026-04-25*
