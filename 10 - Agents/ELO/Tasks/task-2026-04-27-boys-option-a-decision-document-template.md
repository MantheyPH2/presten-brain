---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-26
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Decision Document.md"
---

# Task: Boys Option A — Decision Document Template

## Context

Boys Option A (Boys GA calibration spot check) runs April 29 – May 5. ELO committed to filing a Decision Document within 48 hours of receiving query results from Presten. That document does not exist yet, even as a template.

The Decision Document is what SENTINEL uses to authorize (or delay) Boys club rankings and confirm the Boys calibration path (Option A = current GA cal=100 is viable; or trigger the fallback). Writing it now — before results are in — ensures ELO can file within 48 hours of Presten running the 3 queries, rather than spending that 48-hour window designing the document.

Pre-writing also forces ELO to commit to pass/fail criteria before results are known. Option A PASS and Option A FAIL have different downstream consequences (Boys club rankings proceed vs. Girls-only fallback spec activates). Those consequences are known now. ELO should state them before seeing the results.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Decision Document.md`

Mark this document: `[Template — ELO fills after receiving Boys Option A query results from Presten. Result fields blank until then.]`

---

### Section 1: Context and Criteria

One paragraph: what is Boys Option A testing? What is the concern it resolves?

State explicitly:
- The concern: Boys GA calibration (cal=100) has not been Brier-validated. Girls GA was found to be overcalibrated (cal=100 → causing 40+ pt distortion). Is Boys GA similarly miscalibrated?
- The test: compare Boys vs. Girls rating distributions for the same age group and game volume profile. If Boys GA teams and Girls GA teams of similar competitive level show ratings within a reasonable range of each other (same cal value = similar output), Option A holds. If Boys GA teams show a systematic divergence not explained by competitive level, the cal value needs adjustment.
- Pass threshold: Boys vs. Girls rating divergence < [ELO states specific threshold] points for all age groups tested.
- Fail threshold: Divergence > [threshold] for any age group = Option A fails.

---

### Section 2: Query Results (ELO fills after Presten runs)

**Query 1 — Boys vs. Girls Rating Divergence by Age Group**

| Age Group | Boys GA Avg Rating | Girls GA Avg Rating | Delta | Within Threshold? |
|-----------|--------------------|---------------------|-------|-------------------|
| U13 | [ELO fills] | [ELO fills] | [ELO fills] | PASS / FAIL |
| U14 | [ELO fills] | [ELO fills] | [ELO fills] | PASS / FAIL |
| U15 | [ELO fills] | [ELO fills] | [ELO fills] | PASS / FAIL |
| U16 | [ELO fills] | [ELO fills] | [ELO fills] | PASS / FAIL |
| U17 | [ELO fills] | [ELO fills] | [ELO fills] | PASS / FAIL |

**Query 2 — Boys GA Game Volume Check**

| Age Group | Boys GA Games (count) | Girls GA Games (count) | Volume Parity? |
|-----------|-----------------------|------------------------|----------------|
| U13 | [ELO fills] | [ELO fills] | ADEQUATE / LOW |
| U14 | [ELO fills] | [ELO fills] | ADEQUATE / LOW |
| U15 | [ELO fills] | [ELO fills] | ADEQUATE / LOW |
| U16 | [ELO fills] | [ELO fills] | ADEQUATE / LOW |
| U17 | [ELO fills] | [ELO fills] | ADEQUATE / LOW |

Note: "ADEQUATE" = game count high enough that the rating comparison is statistically meaningful. ELO defines the minimum game count per age group for a valid comparison (e.g., ≥ 50 games per age group). If volume is LOW, flag the comparison as unreliable for that age group.

**Query 3 — Boys GA Top-Team Sanity Check**

| Field | Value |
|-------|-------|
| Highest-rated Boys GA team | [ELO fills: team name, rating, age group] |
| That team's league profile | [ELO fills: is this plausible? Would this team be expected to rank near the top?] |
| Lowest-rated Boys GA team | [ELO fills] |
| Distribution shape (skewed / normal / bimodal) | [ELO fills] |

---

### Section 3: Analysis

One to three paragraphs. ELO writes this section after receiving results.

- Is the Boys GA distribution consistent with cal=100 being appropriate, or does it show signs of the same overcalibration pattern found in Girls GA?
- Are there age groups where Boys GA diverges significantly from Girls GA in ways not explained by competitive level differences?
- Recommendation: does ELO endorse Option A (Boys GA cal=100 holds) or recommend a cal adjustment?

---

### Section 4: Verdict and Downstream Actions

Pre-written decision branches — ELO marks the active branch and fills in specifics.

**Option A PASS (all Query 1 deltas within threshold)**:

```
[ ] ACTIVE — Option A PASS

Boys GA cal=100 is viable. No pre-DSS calibration change required for Boys.

Downstream actions:
[ ] Boys club rankings proceed per full implementation spec (both genders)
[ ] DSS Feature Readiness Tracker: Boys club rankings cell = ✅
[ ] Post-DSS Calibration Roadmap: Boys Option A path active (Boys Brier validation May 17–30)
[ ] Girls-only fallback spec: NOT activated
[ ] SENTINEL notified: "Boys Option A PASS — Boys club rankings authorized for DSS"
```

**Option A FAIL (any Query 1 delta exceeds threshold)**:

```
[ ] ACTIVE — Option A FAIL

Boys GA cal=100 is not viable. Pre-DSS calibration change is [required / deferred].

Failing age groups: [list]
Delta magnitude: [values]

Downstream actions:
[ ] DSS Feature Readiness Tracker: Boys club rankings cell = ❌ (Boys Option A FAIL)
[ ] Girls-only fallback spec ACTIVATED — see Rankings/Club Rankings — Girls-Only Fallback Spec.md
[ ] FORGE notified of scope change: Girls-only implementation proceeds
[ ] Post-DSS Calibration Roadmap: Boys Option B path active (recalibration schedule TBD)
[ ] SENTINEL notified: "Boys Option A FAIL — Girls-only club rankings for DSS. Boys path deferred."
[ ] If FAIL margin is severe (delta > 2× threshold): ELO files incident note in Queue for Presten review
```

---

### Section 5: Filing Instructions

When results are received from Presten:
1. ELO copies Query 1/2/3 output into Section 2 tables
2. ELO writes Section 3 analysis (can be brief — key finding + reasoning)
3. ELO marks the active branch in Section 4 and completes downstream action checklist
4. ELO includes in next briefing: "Boys Option A Decision Document filed. Verdict: [PASS/FAIL]. [Key finding in one sentence]. Downstream actions: [list]."
5. SENTINEL receives this document and confirms FORGE notification if Option A fails.

Target filing time: within 48 hours of receiving Presten's query output.

---

## Quality Standard

- Pass/fail threshold in Section 1 must be a number ELO commits to before seeing results. "Reasonable divergence" is not a criterion.
- Section 3 analysis may be brief (3–5 sentences) but must state ELO's actual conclusion, not a hedge.
- The active branch in Section 4 must be marked unambiguously — not "probably PASS" or "leaning toward FAIL."

## Why This Matters

Boys Option A is the gate for Boys club rankings in the DSS demo. If it fails and FORGE doesn't know until after May 9, the Girls-only fallback timeline becomes critically compressed. Pre-writing the Decision Document ensures ELO can deliver the verdict within 48 hours of results, FORGE can start the fallback implementation (if needed) within the May 5–16 window, and SENTINEL can accurately brief Presten on the DSS scope before May 9. The 30-minute cost of writing this template now avoids a 48-hour design delay under time pressure.

## References

- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — the 3 queries whose results this document analyzes
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — activated if Option A fails
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration background
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys club rankings cell this verdict determines
- `Rankings/Post-DSS Calibration Roadmap — June 2026.md` — active path depends on this verdict
