---
title: MLS NEXT Boys Tier Split — April 28 Decision
type: scheduling-decision
author: ELO
date: 2026-04-24
status: decided
decision: DEFER to May 18–20
related: "[[MLS NEXT Tier Split — Calibration Spec 2026-04-24]]", "[[MLS NEXT Tier Split — Calibration Application Spec]]"
tags: [mls-next, calibration, boys, tier-split, scheduling]
---

# MLS NEXT Boys Tier Split — April 28 Decision

> **Decision: OPTION B — DEFER to May 18–20.** The calibration spec and application spec are complete and ready. Scheduling constraint: FORGE's event classifier has not been run and is a prerequisite for the UPDATE. April 28 session is already scoped (GA ASPIRE fix + ECNL spec review). Apply the MLS NEXT split in the designated May 18–20 calibration window.

---

## Section 1: What the Tier Split Changes

The MLS NEXT Boys tier split replaces the unified `mlsnext = 160` calibration with three distinct values:

| Tier | Cal Value | Change | Affected Teams |
|------|-----------|--------|----------------|
| `mlsnext_homegrown` | **160** | No change | 30 MLS academies + 122 Elite clubs |
| `mlsnext_academy` | **135** | −25 from 160 | 220+ Academy Division clubs |
| `mlsnext_unclassified` | **147** | N/A (new fallback) | ~4–6% of events unclassified by pattern |

**Academy-heavy teams** (U15B, U16B, U17B primarily) are currently over-rated by approximately 20–40 points due to the 160 calibration treating them as Homegrown-quality competitors. Post-split, their ratings correct to reflect their actual cross-league win rate (58.2% vs ECNL Boys).

**Girls rankings:** Not affected. MLS NEXT is a Boys-only league.

Full derivation: [[MLS NEXT Tier Split — Calibration Spec 2026-04-24]]

---

## Section 2: April 28 Feasibility Check

### Q1: Does the GA ASPIRE fix create a sequencing dependency?

No. The GA ASPIRE fix targets `event_tier = 'ga'` events (Girls Academy). The MLS NEXT tier split targets `event_tier = 'mlsnext'` events (Boys). These are completely separate event pools. If both were executed on April 28, the order would be:

1. GA ASPIRE UPDATE (`ga` → `ga_aspire`)
2. Ranking recompute after GA ASPIRE fix
3. MLS NEXT UPDATE (`mlsnext` → `mlsnext_academy` / `mlsnext_homegrown` / `mlsnext_unclassified`)
4. Second ranking recompute after MLS NEXT split

Two ranking recomputes in one session is technically feasible. Sequencing is not a blocker. However — see Q2.

### Q2: How much additional Presten time does the tier split add?

FORGE's `MLS NEXT Event Classifier — Queries 2026-04-24.md` has not been run. Running the classifier is a hard prerequisite for the UPDATE (Section 1.1 of the Application Spec requires confirming ≥90% classification quality before any events are reclassified). The full MLS NEXT session would require:

| Step | Estimated Time |
|------|---------------|
| FORGE Step A: classifier quality check | 5 min |
| Steps B–E: classify and UPDATE events | 20–30 min |
| Update constants file in `compute-rankings.js` | 10 min |
| Full ranking recompute | 10–20 min |
| Post-split validation (Application Spec Section 3 queries) | 15 min |
| **Total** | **60–80 min** |

This nearly doubles the April 28 session length beyond its current scope. It exceeds the ≤30-minute threshold for a same-session addition.

### Q3: Is the Boys MLS NEXT Academy calibration validated?

**Yes — strongly validated by cross-league win-rate data.**

The Academy value of 135 is independently confirmed by two methods:
- Academy vs ECNL Boys: 58.2% win rate (n=4,110) → implies cal range 133–138
- Homegrown vs Academy direct: 66.8% win rate (n=1,680) → 25-point gap → 160 − 25 = 135

Both methods converge at 135 with total sample sizes of ~5,800 games. This is a stronger empirical basis than the Boys GA calibration (which is currently unvalidated — different analysis). The Boys calibration uncertainty raised in [[Boys Calibration Gap Analysis — April 2026]] does NOT apply here.

**Calibration validation is not a blocker.**

---

## Section 3: Decision

**OPTION B — Defer to May 18–20**

The calibration spec and application spec are complete. The calibration values are empirically validated. There is no accuracy reason to delay.

Two practical constraints rule out April 28:

**1. Classifier prerequisite not ready.** FORGE's event classifier has not been run against the production DB. Running the full classifier session — confirming quality, classifying events, applying the UPDATE — adds 60–80 minutes on top of an already-scoped April 28 session.

**2. Application spec designates May 18–20.** This timing was deliberate. The MLS NEXT split shifts Boys ratings for 220+ Academy Division clubs by 20–40 points. Applying this before DSS (May 16) creates visible ranking changes that could affect the demo comparison against USA Rank. The post-DSS window is correct.

**Re-open condition:** Start the May 18 session with the Application Spec pre-flight checklist. FORGE should run the classifier quality check (Step A) before May 18 so the unclassified percentage is known before the calibration session begins.

---

## Section 4: Reference Card Addition

Not applicable — decision is DEFER.

The April 28 Escalation Reference Card (`Product & Planning/April 28 Escalation Reference Card.md`) covers two items: (1) ECNL spec review, (2) GA ASPIRE fix. MLS NEXT tier split is NOT added.

---

## Summary

**MLS NEXT Boys tier split: DEFERRED to May 18–20.** Calibration is validated from empirical win-rate data (4,000+ game samples). Session time and classifier-readiness constraints on April 28 make inclusion impractical. Apply per the Application Spec's designated window. No April 28 Reference Card update needed.

---

## References

- [[MLS NEXT Tier Split — Calibration Spec 2026-04-24]] — calibration values and full derivation
- [[MLS NEXT Tier Split — Calibration Application Spec]] — application procedure, prerequisites, rollback
- [[MLS NEXT Event Classifier — Queries 2026-04-24]] — FORGE's classifier (must run Step A before May 18)
- `02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md` — current session scope (MLS NEXT not included)
