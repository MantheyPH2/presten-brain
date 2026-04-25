---
title: Club Rankings — Calibration Pre-Assessment
type: calibration-assessment
author: ELO
date: 2026-04-24
status: complete
related: "[[Club Rankings — Implementation Plan]]", "[[Calibration Production Runbook — May 2026]]"
tags: [club-rankings, calibration, sequencing, dss, evo-draw]
---

# Club Rankings — Calibration Pre-Assessment

> **Summary:** GA ASPIRE fix (April 28) is a **material dependency** — run before the club rankings session. U12/U13/U14/U19 calibration (May 18–20) is post-DSS and acceptable to defer. **Recommended club rankings implementation date: May 2–3** (after GA ASPIRE fix stabilizes). Demo risk: low for Girls once GA ASPIRE fix is applied; medium for Boys (unvalidated Boys calibration is a known gap, not a new one).

---

## Section 1: GA ASPIRE Dependency

### Which age groups are affected?

Girls GA ASPIRE events exist from U13G through U17G (launched February 2025, primarily targeting the U14G–U17G competitive window). Based on the GA Coverage Audit (`Reports/ga-coverage-audit-results-2026-04-23.md`), approximately 350–1,050 teams participated in GA ASPIRE events and are currently tagged with `event_tier = 'ga'` (cal=140) when the correct value is `event_tier = 'ga_aspire'` (cal=100).

### Expected direction and magnitude of rating change

When the April 28 UPDATE executes and the full ranking recompute runs:

- **Teams primarily in GA ASPIRE events:** ratings decrease approximately **−30 to −50 points** (the 40-point calibration reduction × the fraction of their games in ASPIRE events)
- **Teams with mixed GA ASPIRE + GA core game histories:** ratings decrease approximately **−15 to −30 points**
- **Non-GA-ASPIRE teams:** no change

### Impact on club aggregate rating

A club with multiple Girls GA ASPIRE teams (typical for large NE/NJ programs with 3–5 girls teams in the U13–U17 window) could see their top-team ratings drop 20–40 points per age group. The club aggregate rating (average of top team rating per age group) would drop by roughly:

```
Club rating impact ≈ (number of ASPIRE age groups / total qualifying age groups) × avg_rating_drop_per_age_group
```

For a club with 4 of 7 qualifying age groups in GA ASPIRE events: `(4/7) × 30 = ~17 points` average club rating drop.

### Is this a material rating shift?

**Yes — this exceeds the 30-point materiality threshold.**

A club whose top Girls teams are ASPIRE-heavy could see a 30–50 point club rating drop when the fix applies. If club rankings are computed before April 28, a top-20 Girls club could drop 3–8 positions after the fix. Showing club rankings at DSS that shift visibly within days of the demo would undermine the accuracy claim.

**Conclusion: Club rankings must NOT be computed until after the April 28 GA ASPIRE fix and full recompute.**

---

## Section 2: U12/U13/U14/U19 Calibration Dependency

### The proposed changes (Calibration Production Runbook — May 2026)

| Age Group | Parameter | Current | Proposed |
|-----------|-----------|---------|----------|
| U12 | RD floor | 50 | 60 |
| U13 | K-factor | 32 | 24 |
| U13 | RD floor | 50 | 75 |
| U14 | K-factor | 32 | 28 |
| U14 | RD floor | 50 | 65 |
| U19 | K-factor | 32 | 24 |

### Would these changes cause a material shift in club rankings?

Yes — U12, U13, and U14 are core club ranking age groups (3 of 7 qualifying age groups). K-factor reductions produce smoother rating convergence. For the typical top-ranked team:

- **K-factor reduction (U13: 32→24):** Teams with long dominant histories rise slightly (smoother convergence = less reversion); teams with recent short winning streaks fall slightly. Net effect on club aggregate ratings: **±10–20 points** for clubs with ECNL/GA U13 teams.
- **RD floor increase:** Ratings become slightly less volatile. Net effect: smaller rating swings between club rankings updates. Direct impact on aggregate ratings: **5–15 points** for typical top-club profiles.

**Total estimated club rating shift from U13/U14 calibration changes: 15–35 points for the most affected clubs.**

### Is the May 18–20 timing a problem for DSS?

The DSS demo is May 14–16. Club rankings will be implemented May 2–3. The calibration changes run May 18–20 — two days after DSS concludes. The demo shows club rankings computed under pre-calibration values. Those rankings will shift after DSS when calibration runs.

**This gap is acceptable** for two reasons:
1. The GA ASPIRE fix (April 28) — the larger and more structurally incorrect calibration error — will have been applied before club rankings are built. The remaining calibration changes (K-factor/RD) are precision improvements, not corrections to systematic overcalibration errors.
2. The post-DSS shift is directionally positive — rankings become more accurate. A club director who sees their club at rank #8 at DSS and rank #6 post-calibration is experiencing a correct improvement, not a random fluctuation.

**Recommendation:** Accept the May 18–20 gap. Do not rush the calibration changes into the pre-DSS window to avoid post-DSS shifts — the risk of applying miscalibration right before DSS is higher than the risk of post-DSS rankings updating.

---

## Section 3: Ordering Recommendation

```
Recommended Sequence:

1. GA ASPIRE fix + full ranking recompute (April 28)
   Why: Material dependency — GA ASPIRE overcalibration corrupts Girls club
   rankings by 30–50 points for ASPIRE-heavy clubs. Must run first.
   Owner: Presten (per April 28 Reference Card + ga-aspire-tier-fix spec)

2. Wait 24 hours for rating stabilization (April 29)
   Why: Ensure nightly pipeline has processed the recompute before
   querying the ratings used in club ranking computation.

3. Club Rankings implementation session (May 2–3)
   Why: Calibration is clean post-GA-ASPIRE fix. U12/U13/U14/U19
   calibration is post-DSS and acceptable to defer. 8–12 hour session.
   Owner: Presten

4. DSS demo (May 14–16)
   Club rankings reflect: post-GA-ASPIRE-fix, pre-K-factor-RD-change ratings.
   Most accurate version achievable before DSS.

5. U12/U13/U14/U19 calibration changes (May 18–20)
   Why: Post-DSS precision improvements. Applied after rankings are no
   longer being used for active seeding.
   Owner: Presten (per Calibration Production Runbook)

6. Club rankings recompute after calibration (May 20–21)
   Why: Update club rankings to reflect post-calibration ratings.
   Add to nightly pipeline after May 20 recompute completes.
```

---

## Section 4: Demo Risk Assessment

### Girls club rankings

**Risk: Low** — after GA ASPIRE fix runs April 28, Girls ratings will be correctly calibrated for the demo window. The remaining U13/U14 calibration (K-factor/RD) is a precision improvement that doesn't change the overall ordering significantly for well-established top clubs.

**Safe to demo:** Top Girls club rankings should be stable between May 2 (computed) and May 16 (DSS). One full pipeline cycle separates implementation from demo.

### Boys club rankings

**Risk: Medium** — Boys calibration has not been Brier-validated (per [[Boys Calibration Gap Analysis — April 2026]]). The Boys GA calibration (100) and Boys ECNL calibration (120) are theoretically sound but lack empirical Brier score backing. This means Boys club rankings could have a systematic bias that is unknown before the demo.

**Mitigation:** Run Boys Option A validation queries (3 queries, ~15 minutes) as part of the May 2 club rankings pre-flight. If any of the three queries show a warning signal (Query B: Boys avg_rating diverges >100 points from Girls avg_rating for U13; Query C: Boys GA game volume <500), do not show Boys club rankings at DSS. Default to Girls-only club rankings for the demo.

### Age group coverage gaps

**Flag — Boys U13B, U14B source coverage:** FORGE's Source Gap Inventory should be checked before computing Boys club rankings. If Boys game coverage is thin in certain age groups (USL Academy, EDP, NAL are unconfirmed sources), Boys U13–U15 club rankings may have fewer teams qualifying than Girls equivalents. A Boys club rankings table with only 10–20 qualifying clubs is not demo-ready.

**Safe age group to demo:** Girls U13G–U15G club rankings. These have the deepest source coverage (ECNL, GA, ECNL RL all well-covered for Girls).

**Defer or disclose:** Boys club rankings, U16G–U17G club rankings (ASPIRE complexity warrants checking before demo).

---

## References

- [[Club Rankings — Implementation Plan]] — implementation session guide; start by May 2
- [[Calibration Production Runbook — May 2026]] — U12/U13/U14/U19 changes; May 18–20 window
- `02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md` — GA ASPIRE UPDATE + post-fix verification
- [[Boys Calibration Gap Analysis — April 2026]] — Boys calibration status; Option A queries for pre-flight check
- [[Boys Calibration Option A — Interpretation Guide]] — how to interpret Boys validation queries
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo Spec — May 2026.md` — demo scope and club ranking requirements
