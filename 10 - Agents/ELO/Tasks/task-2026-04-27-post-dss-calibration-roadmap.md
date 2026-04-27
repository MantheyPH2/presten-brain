---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: medium
due: 2026-05-10
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Post-DSS Calibration Roadmap — June 2026.md"
---

# Task: Post-DSS Calibration Roadmap — June 2026

## Context

ELO has three confirmed calibration changes that are blocked from deployment until after DSS (May 16):

1. **U13/U14 calibration fix** — k and rd_floor changes for U12, U13, U14, U19. Affects 6,310+ teams. Do NOT deploy before May 17. Code change + recompute required.
2. **Boys Option A application** — If Option A passes the April 29–May 5 spot check, there are Boys calibration parameters to apply. If it fails, further investigation is needed before any Boys changes.
3. **June 1 age transition risk** — Teams "age up" on June 1 (U14 → U15, etc.). This annually resets some rating contexts and may create anomalies if calibration is mid-change at that time.

No document currently sequences these three changes against each other. The risk: on May 17, ELO has three calibration items in the queue and no agreed-on order. Deploying the U13/U14 fix and the Boys calibration change in the wrong order may complicate validation (which ratings changed because of which fix?). The June 1 transition compounds this — if both fixes are mid-deploy when June 1 hits, ELO cannot cleanly attribute rating changes to calibration vs. age transition.

This task: write the sequenced post-DSS calibration roadmap.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Post-DSS Calibration Roadmap — June 2026.md`

---

### Section 1: Changes in Queue

List all confirmed post-DSS calibration changes. For each:

| Change | Scope | Parameters | Code Change? | Validation Spec |
|--------|-------|-----------|-------------|----------------|
| U13/U14 fix | 6,310+ teams across U12, U13, U14, U19 | k=24 (U13), k=28 (U14), rd_floor adjustments | Yes — compute-rankings.js | `Rankings/Calibration Fix Validation Results.md` |
| Boys Option A (if pass) | Boys age groups with calibration gaps | TBD from spot check results | TBD | `Rankings/Boys Option A Decision Document.md` |
| Boys Option A (if fail) | Investigation scope TBD | N/A | N/A | N/A — investigation phase |

ELO fills in the Boys Option A row after April 29 spot check results are received.

---

### Section 2: June 1 Transition Risk Assessment

What happens to calibration state on June 1?

- Which age groups transition (U13 → U14, U14 → U15, etc.)?
- Does the Evo Draw system handle this transition automatically, or does Presten run a migration step?
- What is the impact on RD (rating deviation) values — do teams transitioning age groups carry their RD forward or reset?
- **Key question for ELO:** If the U13/U14 fix is deployed on May 20 and age transition runs June 1, does the 11-day window produce clean validation data, or does the transition create confounding rating movement?

ELO answers this based on its knowledge of the rating computation logic and the June 1 transition spec (`task-2026-04-25-age-group-transition-june1-risk.md`).

---

### Section 3: Recommended Deployment Sequence

Given the three changes and the June 1 constraint, ELO recommends a sequenced plan:

**Option A sequence (if Boys Option A passes by May 9):**

| Date | Action | Validation window |
|------|--------|------------------|
| May 17 | Deploy U13/U14 fix; run recompute | 3–5 days |
| May 20–22 | Validate U13/U14 results; confirm within expected shift range | — |
| May 23 | Deploy Boys Option A calibration changes; run recompute | 3–5 days |
| May 26–28 | Validate Boys changes; confirm no U13/U14 interference | — |
| June 1 | Age transition runs on stable, validated calibration state | — |

**Option B sequence (if Boys Option A fails by May 9):**

| Date | Action |
|------|--------|
| May 17 | Deploy U13/U14 fix; run recompute |
| May 20–22 | Validate U13/U14 results |
| May 23+ | Begin Boys investigation (do not change Boys calibration yet) |
| June 1 | Age transition on U13/U14-fixed + Boys-unchanged state |

ELO states which option it recommends and why. If ELO believes the sequence should differ from the above, state the alternative and the reasoning.

---

### Section 4: Validation Criteria for Each Change

For each change in the queue, state the specific validation criteria that mark it as "complete and confirmed":

**U13/U14 fix validation:**
- Reference `Rankings/Calibration Fix Validation Results.md` (template exists)
- State the 2–3 key metrics ELO checks to confirm the fix worked as intended
- State the threshold for "looks wrong — investigate before proceeding to next change"

**Boys calibration validation (Option A pass):**
- Reference `Rankings/Boys Option A Decision Document.md`
- State what ELO expects Boys ratings to look like after the change (direction, magnitude)
- State the go/no-go for the Boys change

---

### Section 5: Dependencies and Open Questions for Presten

List items that require Presten's input or execution before the roadmap can be executed:

1. Boys Option A spot check results (expected by May 5) — drives which sequence to follow
2. Code change to `compute-rankings.js` for U13/U14 parameters — requires Presten to apply the change
3. June 1 transition mechanism — confirm whether it is automatic or requires a manual Presten step

For each: who provides the input, when it is needed by, and what decision it unlocks.

---

## Why This Matters

May 17 without this roadmap means ELO and FORGE field "what do we deploy first?" as a real-time question after DSS. With three active calibration changes and a June 1 transition on the horizon, that improvised answer could produce a validation mess: interleaved changes with overlapping effects and no clean baseline between them. This roadmap is written now, while ELO has the context, so May 17 starts with a clear sequence rather than a design meeting.

## Definition of Done

- File written at the deliverable path
- Section 1 lists all confirmed post-DSS calibration changes (Boys row marked "TBD pending Option A results" is acceptable)
- Section 2 answers the June 1 risk question with ELO's actual position (not "TBD")
- Section 3 provides two sequenced options (A and B), each a complete table
- Section 4 has explicit validation criteria for U13/U14 fix (Boys can remain TBD pending Option A)
- Section 5 lists every Presten dependency with a "needed by" date

## References

- `task-2026-04-23-u13-u14-calibration-fix.md` — calibration parameters and code change spec
- `Rankings/Calibration Fix Pre-Deploy Checklist.md` — pre-conditions for deploying U13/U14 fix
- `Rankings/Boys Option A Decision Document.md` — Boys calibration decision gate
- `task-2026-04-25-age-group-transition-june1-risk.md` — June 1 transition risk register
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — monitoring framework for post-change stability
