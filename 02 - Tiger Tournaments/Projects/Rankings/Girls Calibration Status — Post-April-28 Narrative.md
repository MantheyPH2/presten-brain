---
title: Girls Calibration Status — Post-April-28 Narrative
type: calibration-narrative
author: ELO
date: 2026-04-26
status: template
tags: [calibration, ga-aspire, girls, april-28, narrative, evo-draw]
related: "[[April 29 Decision Tree — Verdict Filing Document]]", "[[Post-April-28 Rating Shift Analysis Spec]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[Post-Fix Calibration Stability Monitoring Spec]]"
---

# Girls Calibration Status — Post-April-28 Narrative

> **Status: TEMPLATE — fill in on April 29 after G2 analysis**
>
> Section 1 is pre-written. Sections 2–6 are data entry placeholders. Fill them during the April 29 analysis session — same session as the April 29 Decision Tree — Verdict Filing Document.

---

## Header

```
Status: TEMPLATE — fill in on April 29 after G2 analysis
April 28 Session Date: ___________
April 29 Analysis Run By: ELO
Data source: Post-April-28 recompute
Analysis session start: ___________
Analysis session end: ___________
```

---

## Section 1: What Changed on April 28

The April 28 DB session applied two changes affecting Girls ratings:

### Change 1: GA ASPIRE Reclassification

All historical games with `event_tier = 'ga'` belonging to GA ASPIRE events were updated to `event_tier = 'ga_aspire'`. This re-weights these games in the Elo/Glicko calculation:

- **Before fix:** GA ASPIRE events carried calibration value = **140** (Girls GA cal)
- **After fix:** GA ASPIRE events carry calibration value = **100** (Girls GA ASPIRE cal)
- **Effect:** A reduction of 40 cal points per GA ASPIRE game for Girls teams

In Glicko-2, a higher calibration value inflates the expected win probability for the stronger team against weaker opponents. When GA ASPIRE was tagged at cal=140, the system expected GA ASPIRE teams to dominate lower-tier opponents by a wide margin — artificially suppressing GA ASPIRE team ratings because wins yielded little Glicko credit (the system treated them as "expected"). After the fix at cal=100, expected margins are more modest and wins are rewarded appropriately.

**Expected direction:** Girls GA ASPIRE team ratings should **increase** after the fix. Teams with the most GA ASPIRE game exposure will see the largest increase (estimated +20 to +40 pts for teams with ≥10 GA ASPIRE games). Teams with zero GA ASPIRE exposure should be unchanged.

### Change 2: ECNL Verified Backfill (Step 2b)

TGS and TGS AthleteOne ECNL games received `ecnl_verified = TRUE`. This is a data quality marker used by ECNL migration logic. **No direct rating impact expected** — the `ecnl_verified` flag does not enter the Glicko computation. It is recorded here for completeness.

---

## Section 2: Observed Rating Shifts (fill April 29)

ELO fills this section using the queries from `Rankings/Post-April-28 Rating Shift Analysis Spec.md`.

**Girls GA ASPIRE — Average Rating by Age Group:**

| Age Group | Pre-Fix Avg | Post-Fix Avg | Delta | Predicted Delta (Pre-Run Model) | Matches Prediction? |
|-----------|------------|-------------|-------|----------------------------------|---------------------|
| U13G | | | | +5 to +15 pts (cohort avg) | YES / NO |
| U14G | | | | +8 to +18 pts (cohort avg) | YES / NO |
| U15G | | | | +6 to +15 pts (cohort avg) | YES / NO |
| U16G | | | | +4 to +12 pts (cohort avg) | YES / NO |
| U17G | | | | +2 to +8 pts (cohort avg) | YES / NO |

*Pre-fix avg = values from `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md`. Post-fix avg = values from recompute output. Delta = post minus pre.*

**Girls Non-GA-ASPIRE — Average Rating Spot Check (2 age groups):**

| Age Group | Pre-Fix Avg | Post-Fix Avg | Delta | Note |
|-----------|------------|-------------|-------|------|
| U13G | | | | Expected: ≤ ±3 pts (indirect opponent-rating effect only) |
| U14G | | | | Expected: ≤ ±3 pts |

**Boys Avg Rating Change (diagnostic — should be zero):**

| Age Group | Pre-Fix Avg | Post-Fix Avg | Delta | Within ±5 pt threshold? |
|-----------|------------|-------------|-------|------------------------|
| U13B | | | | YES / NO |
| U14B | | | | YES / NO |
| U15B | | | | YES / NO |

---

## Section 3: Calibration Verdict (fill April 29)

*ELO writes one paragraph answering: Is Girls calibration now correct? Use exactly one of the three defined states.*

**Verdict state (select one):**
- [ ] CONFIRMED IMPROVED — shifts in predicted direction, within acceptable range
- [ ] CONFIRMED CORRECT — no issues found, calibration consistent with expectation
- [ ] STILL UNCERTAIN — one or more age groups show unexpected shifts requiring further analysis

**Paragraph (ELO writes April 29):**

> _GA ASPIRE shifts: [direction and magnitude relative to prediction]_
>
> _Age groups with unexpected shifts: [list or "none"]_
>
> _Non-GA ASPIRE teams: [unchanged / minor adjustment / unexpected movement]_
>
> _Overall: Girls calibration is [CONFIRMED IMPROVED / CONFIRMED CORRECT / STILL UNCERTAIN] as of April 29._

---

## Section 4: Open Issues (fill April 29)

If any age group in Section 2 shows unexpected behavior, list it here with a specific next-step recommendation. If all results are within predicted ranges, write "No open issues — Girls calibration cleared."

| Issue | Age Group | Description | Recommended Action |
|-------|-----------|-------------|-------------------|
| | | | |

---

## Section 5: Impact on Event Strength Authorization (fill April 29)

*One sentence for SENTINEL — copy-paste into Part B of the April 29 Decision Tree — Verdict Filing Document.*

"Girls calibration post-April-28 is [CONFIRMED IMPROVED / CONFIRMED CORRECT / STILL UNCERTAIN]. Event Strength Phase 1 authorization is [CLEAR / CONDITIONAL / BLOCKED] from a calibration standpoint."

---

## Section 6: DSS Readiness Statement (fill April 29)

*One sentence for Presten's DSS preparation notes.*

"As of April 29, Girls GA ASPIRE rankings are [calibrated correctly / showing [X] delta vs. pre-fix baseline], [suitable / not yet suitable] for DSS presentation."

---

## References

- [[April 29 Decision Tree — Verdict Filing Document]] — gate G2 data source; Section 5 feeds Part B SENTINEL Disposition
- [[Post-April-28 Rating Shift Analysis Spec]] — Section 2 query methodology
- [[Post-April-28 Expected Rating Shift — Pre-Run Model]] — Section 2 predicted delta column source
- [[Post-Fix Calibration Stability Monitoring Spec]] — ongoing monitoring after this snapshot
- [[Pre-April-28 Baseline Snapshot — Presten Execution Package]] — pre-fix avg values for Section 2

*ELO — 2026-04-26 (template); filled April 29*
