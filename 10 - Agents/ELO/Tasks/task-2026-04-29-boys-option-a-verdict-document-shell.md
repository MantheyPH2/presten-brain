---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: 2026-04-30 EOD
status: completed
completed: 2026-04-29
topic: boys-option-a-verdict-document-shell
---

# Task: Boys Option A — Verdict Document Shell

## Context

Boys Option A queries (B-OA-1, B-OA-2, B-OA-3) are ready to execute. Presten may run them in today's or tomorrow's session. When results arrive, ELO needs to file the Boys Option A Decision Document within 48 hours.

The 48-hour window is tight if ELO has to build the document from scratch on receipt. Pre-building the shell collapses the turnaround to under 30 minutes — ELO only inserts the query output into a pre-built analysis framework.

---

## What to Build

Create `Rankings/Boys Calibration — Option A Verdict Document.md`

A complete verdict document with all static content written and all result-dependent cells marked [TO FILL: B-OA-1/2/3 results].

---

## Required Content

### Section 1 — Document Purpose

1 paragraph: what this document decides (whether Boys Option A calibration parameters are approved for deployment), when it was filed, and what happens next based on the verdict.

### Section 2 — Option A Parameters Being Evaluated

State the exact parameters under evaluation (K-factor, RD, age groups). Source from `Rankings/Boys Calibration — Option A Boys April-29 Execution Package.md` or equivalent filed document. Do not re-derive — copy the parameters.

### Section 3 — Query Results

Pre-build the results table:

| Query | Description | Result | Pass Threshold | Pass? |
|-------|-------------|--------|----------------|-------|
| B-OA-1 | Boys rating distribution by age group | [TO FILL] | [ELO fills threshold] | [TO FILL] |
| B-OA-2 | Boys pre-fix baseline for targeted age groups | [TO FILL] | N/A (baseline) | N/A |
| B-OA-3 | Boys post-Option-A projection | [TO FILL] | Better than B-OA-2 AND meets minimum | [TO FILL] |

**ELO pre-fills all thresholds.** The threshold column should be complete before results arrive.

### Section 4 — Analysis

Pre-write the analysis framework — the questions ELO will answer once results are in. Format as fill-in sections:

**Rating distribution health (B-OA-1):**
> [TO FILL: Interpretation of B-OA-1 distribution. Does distribution shape indicate calibration alignment? Notable outliers?]

**Pre-fix vs. post-Option-A comparison (B-OA-2 vs. B-OA-3):**
> [TO FILL: Direction correct? Magnitude acceptable? Any age group where Option A makes things worse?]

**Threshold evaluation:**
> [TO FILL: Does B-OA-3 meet the pass threshold? If conditional path (direction correct, threshold not met): what is ELO's recommendation?]

### Section 5 — Verdict

Pre-write the verdict matrix:

| Verdict | Condition | Next Step |
|---------|-----------|-----------|
| **APPROVE** | B-OA-3 passes threshold for all targeted age groups | Proceed to Boys Brier pre-check May 10–16 per execution package |
| **CONDITIONAL** | Direction correct; threshold not met | ELO files conditional recommendation with revised parameters; SENTINEL review required |
| **REJECT** | B-OA-3 worse than B-OA-2 or inconclusive | FORGE not authorized to deploy May 17; ELO proposes fallback parameters |

**Actual verdict:** [TO FILL: APPROVE / CONDITIONAL / REJECT]

**Verdict rationale:** [TO FILL: 2–3 sentences]

### Section 6 — Implications

Pre-write the downstream impact section:

**If APPROVE:** [ELO fills: what FORGE authorizes, what ELO monitors, when Boys Brier pre-check runs]

**If CONDITIONAL or REJECT:** [ELO fills: fallback path, timeline impact to May 17, SENTINEL notification requirement]

### Section 7 — Filing Record

```
Query results received: [TO FILL: date/time]
Analysis completed: [TO FILL: date/time]
Document filed: [TO FILL: date/time]
Filed by: ELO
```

---

## Deliverable

`Rankings/Boys Calibration — Option A Verdict Document.md`

Shell complete. All static content written. All result-dependent cells pre-formatted with [TO FILL] markers. All pass thresholds pre-filled. Confirm delivery in briefing.

When Presten runs B-OA-1/2/3, ELO inserts results into the pre-built table and writes the analysis. Filing time < 30 minutes from results receipt.
