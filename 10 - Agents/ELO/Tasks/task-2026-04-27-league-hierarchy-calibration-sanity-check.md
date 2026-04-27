---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: high
due: 2026-05-09
deliverable: "02 - Tiger Tournaments/Projects/Rankings/League Hierarchy Calibration Sanity Check — April 2026.md"
---

# Task: League Hierarchy Calibration Sanity Check — April 2026

## Context

The League Hierarchy defines calibration values for every league in Evo Draw. These values have been set and periodically adjusted, but there is no document that audits the full set for internal consistency. SENTINEL's mandate includes verifying that teams are not mis-ranked due to data errors — and miscalibrated leagues are a systematic source of mis-ranking that affects every team in that league.

Two specific concerns that have surfaced:
1. The GA ASPIRE overcalibration (cal=100 when events were tagged as ga/cal=140) was a 40-point error that affected hundreds of teams and was not caught through routine review. The calibration values need a systematic sanity check.
2. New leagues are added to the hierarchy without a cross-validation step against existing leagues. A new league's calibration value may be inconsistent with its actual competitive tier.

This task: audit the full League Hierarchy calibration values for systematic errors, relative inconsistencies, and leagues that warrant further investigation.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy Calibration Sanity Check — April 2026.md`

---

### Section 1: Audit Methodology

Describe the audit approach ELO uses. At minimum:

- **Internal consistency check:** Are leagues that are competitive peers assigned similar calibration values? (ECNL and MLS NEXT should be close; regional leagues should be materially lower than national leagues.)
- **Rating distribution check:** Using available game data, do teams that play primarily in a given league actually rank at the expected level? A league with cal=150 (high) should have teams with above-average ratings.
- **Outlier detection:** Which league has the highest calibration value? Lowest? Are these outliers justified by their competitive position, or are they artifacts of when the value was set?

ELO describes the specific checks it ran, even if some checks are qualitative due to query access limitations.

---

### Section 2: Full Calibration Value Audit Table

List every league in the League Hierarchy with its calibration value. Add two assessment columns:

| League | Cal Value | Expected Tier | Assessment | Flag? |
|--------|-----------|---------------|-----------|-------|
| ECNL | [N] | National Elite | OK / High / Low | Y/N |
| MLS NEXT | [N] | National Elite | OK / High / Low | Y/N |
| GA ASPIRE | [N] | Regional Premier | OK / High / Low | Y/N |
| ... | | | | |

**Assessment values:**
- `OK` — calibration value is consistent with the league's competitive tier relative to peers
- `High` — calibration value appears higher than the league's actual competitive tier warrants; teams in this league may be systematically over-ranked
- `Low` — calibration value appears lower than the league's competitive tier warrants; teams may be under-ranked
- `Insufficient data` — ELO cannot assess without running distribution queries

**Flag** = Yes if ELO recommends SENTINEL review the value with Presten before DSS.

---

### Section 3: Flagged Leagues — Investigation Notes

For each league flagged in Section 2, provide:

- Why the value appears inconsistent (peer comparison, distribution observation, or structural concern)
- What the correct value might be, with reasoning
- What query or data would confirm whether the flag is valid
- Priority: fix before DSS / fix post-DSS / monitor only

---

### Section 4: Relative Tier Consistency Check

Independently of absolute values, check that the calibration values produce a defensible tier hierarchy. For Girls:

- National elite tier (ECNL, MLS NEXT): cal values should be within [N] points of each other
- Regional premier tier (GA, ECNL Regional League): cal values should be materially below national elite
- State-level tier: cal values should be materially below regional premier

Draw the actual tier map using the cal values. If any league's cal value places it in the wrong tier (e.g., a state-level league with a cal value equal to a national league), flag it.

Repeat for Boys if Boys calibration is sufficiently independent to assess.

---

### Section 5: Pre-DSS Risk Assessment

Given the flags from Sections 2–4, state ELO's risk assessment for the DSS demo:

- **Low risk:** All checked leagues are within 15 cal points of their expected tier. No systematic mis-ranking visible.
- **Medium risk:** 1–3 leagues flagged as potentially miscalibrated. Affected teams may be modestly mis-ranked but no demo-breaking anomalies.
- **High risk:** A flagged league has cal value > 30 points off expected tier, or affects a demographic that will be shown in the demo (e.g., U16G if GA is a primary demo age group).

For each High risk flag: state the specific teams or age groups at risk, and what SENTINEL should tell Presten.

---

## Why This Matters

The GA ASPIRE miscalibration showed that a single wrong cal value can distort rankings for hundreds of teams over an extended period before it is caught. The fix was caught because ELO investigated a specific complaint (GA teams ranking anomalously high). A proactive audit catches the same type of error before it surfaces as a complaint or demo failure. With DSS 20 days away, now is the last practical time to identify and fix calibration errors. An error found May 9 can be corrected before DSS. An error found May 15 cannot. This audit is SENTINEL's quality gate on the League Hierarchy.

## Definition of Done

- File written at the deliverable path
- Section 2 is a complete table: every league in the League Hierarchy has a row
- All flags in Section 2 have corresponding investigation notes in Section 3
- Section 4 draws the tier map (even if qualitative) with flagged outliers identified
- Section 5 states a specific risk level (Low / Medium / High) with rationale
- FORGE briefing summary: "League calibration audit complete. [N] leagues flagged. Risk level: [Low/Medium/High]. Highest-priority flag: [league name, reason]."

## References

- [[League Hierarchy]] — source of calibration values being audited
- `Rankings/GA Coverage Audit Results — 2026-04-23.md` — the most recent example of a calibration value error
- `Rankings/Calibration Values — League Hierarchy Reconciliation.md` — filed April 26, context for calibration state
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration context
- `Rankings/DSS Ranking Accuracy Claims Spec.md` — accuracy claims that depend on correct calibration values
