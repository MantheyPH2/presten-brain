---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Girls Calibration Status — Post-April-28 Narrative.md"
---

# Task: Girls Calibration Status — Post-April-28 Narrative Template

## Context

The April 29 Decision Tree Verdict Filing Document captures four gate verdicts (G1–G4). Gate G2 is a pass/fail check on average rating shifts. What G2 does not produce is the calibration narrative — a qualitative interpretation of whether Girls calibration is now correct, directionally improved, or still has open issues.

SENTINEL needs more than "G2 PASS." To authorize Event Strength Phase 1 and to brief Presten, SENTINEL needs to know: what does the post-April-28 Girls calibration state actually look like? Did GA ASPIRE teams move in the expected direction? Are any age groups behaving unexpectedly? Is Girls calibration ready for DSS, or are there open caveats?

Currently this analysis is scattered across: the Pre-Run Model, the Rating Shift Analysis Spec, the Post-Fix Calibration Stability Monitoring Spec, and the Verdict Filing Document. ELO has not synthesized these into a single calibration status document.

This task: ELO writes a template now (pre-April-28) with the structure for the narrative. ELO fills it in on April 29 using G2 data. SENTINEL reads it in the same April 29 session to make the Event Strength authorization decision.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Girls Calibration Status — Post-April-28 Narrative.md`

Write this document as a template now. Sections that can be pre-written (context, methodology, expected outcomes) are filled in now. Data cells are left as placeholders for April 29.

---

### Header

```
Status: TEMPLATE — fill in on April 29 after G2 analysis
April 28 Session Date: ___________
April 29 Analysis Run By: ELO
Data source: Post-April-28 recompute
```

---

### Section 1: What Changed on April 28

Pre-fill now (does not require April 29 data):

The April 28 DB session applied two changes affecting Girls ratings:

1. **GA ASPIRE reclassification:** All historical games with `event_tier = 'ga'` belonging to GA ASPIRE events were updated to `event_tier = 'ga_aspire'`. This re-weights these games in the Elo calculation: GA ASPIRE games now carry a higher calibration value than standard GA events, because GA ASPIRE is Tier 1 (Girls).

2. **ecnl_verified backfill (Step 2b):** TGS and TGS AthleteOne ECNL games received `ecnl_verified = TRUE`. This does not change ratings directly — it is a data quality marker used by ECNL migration logic. No direct rating impact expected.

Expected direction of change: GA ASPIRE teams' ratings should increase (their games are now weighted higher). Non-GA ASPIRE teams playing against GA ASPIRE teams should see a corresponding adjustment. Teams with no GA ASPIRE games should be unchanged.

---

### Section 2: Observed Rating Shifts (fill April 29)

ELO fills this section using G2 queries from the Rating Shift Analysis Spec.

**Girls GA ASPIRE — Average by Age Group:**

| Age Group | Pre-Fix Avg | Post-Fix Avg | Delta | Predicted Delta (from Pre-Run Model) | Matches Prediction? |
|-----------|------------|-------------|-------|--------------------------------------|---------------------|
| U13G | | | | | YES / NO |
| U14G | | | | | YES / NO |
| U15G | | | | | YES / NO |
| U16G | | | | | YES / NO |
| U17G | | | | | YES / NO |

**Girls Non-GA-ASPIRE — Average by Age Group (spot check):**

| Age Group | Pre-Fix Avg | Post-Fix Avg | Delta | Note |
|-----------|------------|-------------|-------|------|
| U13G | | | | Expected: near-zero or small negative |
| U14G | | | | |

---

### Section 3: Calibration Verdict (fill April 29)

ELO writes one paragraph answering: Is Girls calibration now correct?

Structure:
- GA ASPIRE shifts: [direction and magnitude relative to prediction]
- Any age groups with unexpected shifts: [list or "none"]
- Non-GA ASPIRE teams: [unchanged / minor adjustment / unexpected movement]
- Overall assessment: [Girls calibration is CONFIRMED IMPROVED / CONFIRMED CORRECT / STILL UNCERTAIN — one of these three]

Confirmed Improved = shifts in predicted direction, within acceptable range
Confirmed Correct = no issues found, calibration consistent with pre-April-28 expectation
Still Uncertain = one or more age groups show unexpected shifts requiring further analysis

---

### Section 4: Open Issues (fill April 29)

If any age group in Section 2 shows unexpected behavior, list it here with a specific next-step recommendation. If none: write "No open issues — Girls calibration cleared."

| Issue | Age Group | Description | Recommended Action |
|-------|-----------|-------------|-------------------|
| | | | |

---

### Section 5: Impact on Event Strength Authorization (fill April 29)

One sentence for SENTINEL:

"Girls calibration post-April-28 is [CONFIRMED IMPROVED / CONFIRMED CORRECT / STILL UNCERTAIN]. Event Strength Phase 1 authorization is [CLEAR / CONDITIONAL / BLOCKED] from a calibration standpoint."

This sentence feeds directly into SENTINEL's Verdict Filing Document — Part B, Girls calibration status line.

---

### Section 6: DSS Readiness Statement (fill April 29)

One sentence ELO writes for Presten's DSS preparation:

"As of April 29, Girls GA ASPIRE rankings are [calibrated correctly / showing [X] delta vs. pre-fix baseline], suitable / not yet suitable for DSS presentation."

---

## Quality Standard

- Sections 1 is pre-written by April 29 (template work done now)
- Sections 2–6 are filled in during the April 29 analysis session, same session as the Verdict Filing Document
- Section 3 verdict must be one of three defined states — no ambiguous language like "largely correct"
- Section 5 must be a single sentence SENTINEL can copy into the Verdict Filing Document Part B

## Why This Matters

SENTINEL's April 29 authorization decision has two components: the Verdict Filing Document (gate verdicts) and the calibration narrative (is the result actually good?). Gate verdicts answer "did the math check out." The calibration narrative answers "do we believe Girls rankings are now correct." DSS is May 9. If Girls calibration is "still uncertain" on April 29, SENTINEL and Presten need to know 10 days in advance — not on May 8 when it's too late to fix.

## References

- `Rankings/April 29 Decision Tree — Verdict Filing Document.md` — gate G2 data source; Section 5 feeds Part B
- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — Section 2 query methodology
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — Section 2 predicted delta column
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — ongoing monitoring after this snapshot
- `Product & Planning/DSS Demo Spec — May 2026.md` — Section 6 DSS readiness context
