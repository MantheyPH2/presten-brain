---
type: agent-task
assigned_by: SENTINEL
assigned_to: ELO
date: 2026-04-29
priority: medium
due: "2026-05-02"
status: completed
completed: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Tier 2 Leagues — Calibration Scope Decision.md"
topic: tier2-leagues-calibration-scope-doc
---

# Task: Tier 2 Leagues Calibration — Scope and Decision Document

## Objective

Four Tier 2 leagues (USL Academy, Elite 64, EDP, NAL) currently show calibration status as **UNDEFINED** in the Pre-May-9 Calibration State Dashboard. SENTINEL needs a clear document before May 9 DSS explaining the current state, ELO's recommended approach, and what Presten must decide.

## Background

From `Rankings/Pre-May-9 Calibration State Dashboard.md` Section 1:

> "Four Tier 2 leagues (USL Academy, Elite 64, EDP, NAL) remain UNDEFINED pending Presten review."

These leagues appear in the League Hierarchy with calibration values that have not been explicitly validated by ELO. This gap is low urgency for May 9 DSS (they are not primary demo leagues) but represents a completeness gap that SENTINEL must be able to explain to Presten.

Additionally, the May 9 DSS Risk Register may need to reflect this gap in its coverage section.

## Deliverable

File `Rankings/Tier 2 Leagues — Calibration Scope Decision.md` with:

### Section 1 — Current State per League

| League | Current Cal Value in Engine | Source of Cal Value | Validation Status | Games in DB |
|--------|-----------------------------|---------------------|-------------------|-------------|
| USL Academy | [FILL from League Hierarchy] | [League Hierarchy / assumed] | UNVALIDATED / VALIDATED | [FILL if known] |
| Elite 64 | [FILL] | [FILL] | UNVALIDATED / VALIDATED | [FILL if known] |
| EDP | [FILL] | [FILL] | UNVALIDATED / VALIDATED | [FILL if known] |
| NAL | [FILL] | [FILL] | UNVALIDATED / VALIDATED | [FILL if known] |

ELO fills from the League Hierarchy and any prior calibration work. If values are unknown, state "NOT SET — using default engine value."

### Section 2 — ELO's Recommendation

For each league, one of:
- **Use current value as-is** — no Brier evidence yet, but value is reasonable based on [rationale]. Revisit post-June when more games accumulate.
- **Set to Tier 2 default** — use the same cal value as the nearest peer league.
- **Defer to post-May-9** — insufficient games in DB to calibrate. Flag as known gap in DSS demo.
- **Escalate to Presten** — value requires domain judgment ELO cannot make alone.

### Section 3 — DSS Demo Impact

One sentence per league: does this UNDEFINED calibration affect what Presten shows in the May 9 DSS demo? If these leagues are not demo-visible, say so.

Overall DSS impact assessment: **NONE / LOW / MEDIUM / HIGH**

### Section 4 — SENTINEL Decision Request

List any items that require SENTINEL or Presten to decide. If no decision is needed (ELO can set defaults), state that explicitly.

Format:
```
Decision needed from Presten:
[ ] Approve ELO's recommended defaults for all four Tier 2 leagues
[ ] Override specific leagues with different values (specify)
[ ] Defer all Tier 2 calibration to post-May-9 formal review

SENTINEL can authorize by: [date]
```

## Notes

- This document is informational and planning-focused — it does not require DB queries to file a useful draft. ELO can complete a first version from vault documents alone.
- If any Tier 2 league has sufficient games in DB for a Brier analysis, ELO notes that but does not run Brier without SENTINEL authorization.
- Due May 2 — leaves SENTINEL time to review before May 9.
