---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Post-Authorization Runbook.md"
---

# Task: Event Strength Phase 1 — Post-Authorization Runbook

## Context

The April 29 Decision Tree authorizes (or holds) Event Strength Phase 1. If all four gates pass and SENTINEL signs off on the Verdict Filing Document, Event Strength Phase 1 is authorized. The Phase 1 Execution Package (filed April 24) covers pre-authorization analysis and readiness checks. The Phase 1 Authorization Criteria (filed April 25) defines what SENTINEL reviews.

What does not exist: a step-by-step runbook for what ELO actually does on Day 1 after authorization. If April 29 returns AUTHORIZED at 3 PM, what does ELO do at 3:01 PM? What queries run, in what order, and how does ELO confirm Phase 1 is complete? The current documentation answers "should we authorize" — this document answers "what happens after we do."

Without this runbook, authorization → 2-3 day planning delay while ELO reconstructs the implementation sequence. Write this now so April 29 authorization → same-session implementation begins.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Post-Authorization Runbook.md`

---

### Header

```
Status: AWAITING AUTHORIZATION
Authorization Date: ___________ (fill April 29 after Decision Tree)
Authorizing agent: SENTINEL
ELO Execution Lead: ELO
Target completion: Within 48 hours of authorization
```

---

### Pre-Condition Check (read before starting)

Before running any Phase 1 steps, confirm:
- [ ] April 29 Verdict Filing Document — all 4 gates PASS
- [ ] SENTINEL Disposition section signed
- [ ] Girls GA ASPIRE post-fix rating distribution confirmed stable (G2 PASS)
- [ ] No active pipeline issues flagged by FORGE (check latest FORGE briefing)

If any pre-condition is unmet: do not proceed. File status in next ELO briefing and notify SENTINEL.

---

### Step 1: Confirm Event Strength Tier Assignments

**What:** Verify the event_tier → strength multiplier mapping is current and complete before running the Phase 1 recompute.

Reference: `Rankings/Event Strength Phase 1 — Execution Package.md` tier mapping table.

**Action:** Run a quick check:
```sql
SELECT event_tier, COUNT(*) AS game_count
FROM games
WHERE season = '2025-2026'
GROUP BY event_tier
ORDER BY game_count DESC;
```

Cross-check all `event_tier` values against the Phase 1 tier mapping table. Any `event_tier` not in the mapping table → STOP. Add the tier before proceeding. An unmapped tier means those games receive no event strength weight — silent accuracy loss.

**Gate:** All returned `event_tier` values have a defined strength multiplier. File list in this document.

---

### Step 2: Snapshot Pre-Phase-1 Ratings

**What:** Capture average ratings by age group before applying event strength. This is the comparison baseline for the Phase 1 impact analysis.

```sql
SELECT age_group, gender, AVG(rating) AS avg_rating, COUNT(*) AS team_count
FROM teams
WHERE active = TRUE
GROUP BY age_group, gender
ORDER BY gender, age_group;
```

Paste output below as the pre-Phase-1 baseline. This is also the comparison for SENTINEL's Phase 1 impact assessment.

| Age Group | Gender | Avg Rating (Pre-Phase 1) | Team Count |
|-----------|--------|--------------------------|------------|
| | | | |

---

### Step 3: Run Phase 1 Recompute

**What:** Apply event strength weights to the rating calculation and recompute all team ratings.

Reference: `Rankings/Event Strength Phase 1 — Execution Package.md` — the specific SQL or script to run.

ELO does NOT write the implementation query in this document — it is in the Execution Package. This runbook references it. The runbook's job is to ensure this step is run in the right order, not to duplicate the query.

**Action:** Run Phase 1 recompute per Execution Package. Log start time and end time.

**Expected:** All team ratings updated. Total team count unchanged. No teams should disappear or have NULL ratings after recompute.

---

### Step 4: Validate Phase 1 Output

Run the post-Phase-1 validation queries from `Rankings/Event Strength Phase 1 — Execution Package.md`.

Additionally, compare the pre-Phase-1 baseline (Step 2) against the post-Phase-1 state:

```sql
SELECT age_group, gender, AVG(rating) AS avg_rating_post, COUNT(*) AS team_count
FROM teams
WHERE active = TRUE
GROUP BY age_group, gender
ORDER BY gender, age_group;
```

**PASS criteria:**
- Average rating shifts are directionally consistent with `Rankings/Event Strength Phase 1 — Rankings Impact Preview.md` predictions
- No age group shows an average shift > 2x the predicted magnitude
- No teams have NULL or 0 rating post-recompute

**If shift magnitude is > 2x predicted:** Do not file Phase 1 as complete. Flag to SENTINEL with the specific age groups exceeding prediction. This may indicate a multiplier calibration error.

---

### Step 5: File Phase 1 Completion Record

Once Step 4 passes, update this document's frontmatter:
- `status:` → `completed`
- `completed:` → date/time

File an ELO briefing confirming Phase 1 complete with:
- Pre vs. post average ratings (Step 2 and Step 4 tables side-by-side)
- One sentence on whether shifts matched prediction
- Any age groups requiring follow-up

SENTINEL receives this as the Phase 1 completion record.

---

### Step 6: Phase 2 Pre-Authorization Check

After Phase 1 is confirmed complete, ELO reviews the Phase 2 prerequisites (filed April 25) to determine whether Phase 2 can be authorized. Phase 2 is NOT authorized automatically with Phase 1 — it requires a separate SENTINEL sign-off. This is a review step only: does ELO's Phase 1 output meet Phase 2's pre-authorization criteria?

File status in the same briefing as Step 5: "Phase 2 prerequisites met / not met."

---

## Quality Standard

- Steps 1–5 must be completable in a single psql session
- Every gate (Step 1, Step 4) must have a specific pass/fail condition — not "looks reasonable"
- Step 2 baseline table must be filled in before Step 3 runs — not after
- This document must be updated in real time during execution, not reconstructed afterward

## Why This Matters

Event Strength Phase 1 has been the authorization target since April 22. The calibration fix (April 28) is the prerequisite. The Decision Tree (April 29) is the authorization gate. This runbook is the execution. Without it, authorization approval → planning delay. With it, SENTINEL authorizes on April 29 and ELO begins implementation the same day.

## References

- `Rankings/April 29 Decision Tree — Verdict Filing Document.md` — authorization trigger
- `Rankings/Event Strength Phase 1 — Execution Package.md` — Step 3 query source
- `Rankings/Event Strength Phase 1 — Rankings Impact Preview.md` — Step 4 validation comparison
- `Rankings/Event Strength Phase 2 — Prerequisites.md` — Step 6 reference
