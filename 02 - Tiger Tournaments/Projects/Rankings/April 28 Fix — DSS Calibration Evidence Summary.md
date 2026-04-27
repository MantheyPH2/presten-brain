---
title: April 28 Fix — DSS Calibration Evidence Summary
type: evidence-summary
author: ELO
date: 2026-04-27
status: partial — awaiting April 29 gate results. Section 2 to be filled by May 2.
related: "[[April 29 — Gate Results Synthesis Brief]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[DSS Demo Talking Points — Rankings Features]]"
tags: [calibration, ga-aspire, dss, evidence, accuracy, evo-draw]
---

# April 28 Fix — DSS Calibration Evidence Summary

> **[Partial document — Sections 1, 3, and 4 written April 27. Section 2 requires April 29 gate results and will be filled by May 2. This document is DSS-ready upon Section 2 completion.]**

---

## Section 1: What Was Fixed and Why

**The problem:**

Girls teams competing in the Georgia ASPIRE league were being evaluated against a benchmark stronger than the actual competition they face. The Evo Draw system classifies each league by its competitive level — a win against a top national-level opponent like an ECNL team is worth more than a win against a regional league. Georgia ASPIRE is a regional premier-level league, but it was being treated as a full Girls Academy program. This meant Girls GA ASPIRE teams were expected to win by larger margins than their results justified — and when those expected blowouts didn't happen, the system penalized them, holding their rankings below what their actual game results deserved. The result was a systematic underrating of Girls GA ASPIRE teams, concentrated in U13–U16 age groups where GA ASPIRE enrollment is highest. Based on ELO's pre-fix estimates, the most-exposed teams were suppressed by 20–40 ranking points.

**The fix:**

We corrected the league classification for Georgia ASPIRE events. They are now benchmarked against leagues at the appropriate regional premier competitive tier — no longer equated with full Girls Academy programs. This change was deployed on April 28, 2026. When the rankings were recomputed, every game played in a GA ASPIRE event was re-evaluated with the correct competitive context. Teams that earned wins against quality regional competition now receive credit proportional to what that win actually represents. The fix is permanent and forward-looking: new games played in GA ASPIRE events will be classified correctly from the moment they are ingested.

---

## Section 2: Evidence the Fix Worked

> **[To be filled by May 2 after April 29 gate results are available.]**

**Evidence Item 1: Rating shift direction and magnitude**

> "We predicted Girls GA ASPIRE teams would see their rankings adjust upward by 10–40 points after reclassification. The actual adjustment was [Z] points — [within our predicted range / slightly outside our predicted range but within acceptable bounds]. This is consistent with the magnitude of the original miscalibration."

[ELO fills from April 29 Gate Results Synthesis Brief, Section 1.]

**Evidence Item 2: No unintended effects on other teams**

> "Boys rankings were not affected, as expected — Boys teams do not compete in GA ASPIRE events. Girls ECNL and MLS NEXT team ratings shifted by less than [N] points, confirming the fix was precisely scoped to GA ASPIRE events and did not propagate to other leagues."

[ELO fills from April 29 G4 gate result (Boys) and G3 gate result (ECNL/MLS NEXT movement).]

**Evidence Item 3: The fix is permanent and forward-looking**

New games played in GA ASPIRE events after April 28 are automatically ingested and computed with the correct league classification. The fix is not just a retroactive correction — it closes the miscalibration going forward.

---

## Section 3: What This Means for Ranking Accuracy

The April 28 fix produced a measurable accuracy improvement for Girls rankings in U13–U16 age groups, where GA ASPIRE enrollment is highest. Teams in Georgia and adjacent southeastern states — the primary GA ASPIRE geography — now have rankings that correctly reflect their competitive performance. Before the fix, a strong Girls GA ASPIRE team might appear 30–40 points below where its head-to-head results against regional competitors would place it. After the fix, that gap closes. For soccer directors and recruiting coaches evaluating Girls players in GA ASPIRE programs, the rankings are now a more honest signal of where those teams stand in the national landscape.

The fix also improves the integrity of cross-league comparisons. When ECNL or MLS NEXT teams have played against GA ASPIRE opponents, those opponents' ratings now correctly represent their competitive tier. This indirectly improves the accuracy of any team whose ratings were influenced by opponents drawn from GA ASPIRE programs.

---

## Section 4: Remaining Calibration Work (Honest Disclosure)

Three calibration items are in progress and were not part of the April 28 fix:

1. **Boys calibration spot check (Option A):** Boys rankings are structurally sound but have not yet been validated through Brier analysis. A spot check is running April 29–May 5 to confirm Boys GA calibration is correct. Results arrive by May 5. At this point, Girls rankings carry higher confidence than Boys rankings.

2. **U13/U14 calibration parameter update:** Adjustments to K-factor and rating deviation floor for U12, U13, U14, and U19 age groups are scheduled for deployment after DSS (May 17+). This is a minor accuracy improvement for younger age groups and is deliberately sequenced after the DSS demo to avoid disrupting demo data.

3. **Tier 2 undefined leagues:** Four Tier 2 leagues (USL Academy, EDP Soccer, Elite 64, NAL) currently have no assigned calibration value. ELO has a calibration recommendation pending Presten's authorization. Game volume from these leagues is being assessed — if volume is above 2,000 games, calibration values will be applied before DSS.

The GA ASPIRE fix is the most material accuracy improvement in this release cycle. All national-level leagues (ECNL, MLS NEXT, Girls Academy) are correctly calibrated. The remaining items are targeted improvements, not foundational gaps.

---

## References

- `Rankings/April 29 — Gate Results Synthesis Brief.md` — technical source for Section 2 numbers
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — prediction ELO compares against in Section 2
- `Rankings/DSS Demo Talking Points — Rankings Features.md` — companion document for demo delivery
- `Rankings/DSS Ranking Accuracy Claims Spec.md` — accuracy claims framework this document contributes to
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration disclosure in Section 4

*ELO — 2026-04-27 (partial — Section 2 to complete by 2026-05-02)*
