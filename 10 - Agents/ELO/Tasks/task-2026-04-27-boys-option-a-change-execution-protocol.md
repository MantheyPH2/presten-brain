---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-05-04
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Calibration Change Execution Protocol.md"
---

# Task: Boys Option A — Calibration Change Execution Protocol

## Context

The Boys Option A process is well-designed. The query pack is validated. The Pre-Verdict Evidence Checklist enforces the 5-day spot-check window. The Decision Document template is ready. 

What does not yet exist: **the execution protocol for after the verdict is YES.**

If the Boys Option A verdict is YES (filed May 4–5), the next step is to implement the calibration changes. Currently ELO would enter that session knowing only: "Boys Option A passed — now implement the calibration." That is not enough. Without a pre-written execution protocol, ELO faces a design session under time pressure: what exactly changes, in what order, with what verification.

This task: write the execution protocol now, before the verdict, so that if the verdict is YES, ELO can begin execution within the same session (or the immediately following session) rather than scheduling a new design session.

This protocol is conditionally relevant — it only executes if the verdict is YES. But SENTINEL requires it to exist before the verdict, not after.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Calibration Change Execution Protocol.md`

---

### Section 1: Option A — What Changes

Before writing the execution steps, state explicitly what Option A actually modifies. ELO fills this from the Decision Document and query pack:

| Parameter | Current Value | Option A Value | Change Type |
|-----------|--------------|----------------|-------------|
| [Boys calibration parameter 1] | | | |
| [Boys calibration parameter 2] | | | |
| [Affected age groups] | | | |
| [Affected event_tier values] | | | |

If Option A is a K-factor change, state the K-factor. If it is a calibration multiplier, state the multiplier. Be specific — not "adjust Boys calibration" but "change `boys_k_factor` from X to Y."

---

### Section 2: Pre-Execution Authorization Gate

Before executing any calibration change:

- [ ] Boys Option A verdict document is filed with verdict = YES
- [ ] Pre-Verdict Evidence Checklist is complete — all 3 conditions met (queries run, 5-day window, criteria written before results)
- [ ] SENTINEL has been notified of the YES verdict
- [ ] SENTINEL has issued execution authorization (can be same-day as verdict)
- [ ] No active ELO flags that would make Boys calibration changes risky (e.g., April 29 gate failures, active ECNL migration issues affecting Boys ratings)

---

### Section 3: Pre-Execution Baseline Snapshot

Before making any changes, capture the current Boys ratings baseline:

```sql
-- Baseline: Boys rating distribution before Option A
SELECT 
    age_group,
    COUNT(*) as team_count,
    ROUND(AVG(rating), 1) as avg_rating,
    ROUND(PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY rating), 1) as p10,
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating), 1) as median,
    ROUND(PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY rating), 1) as p90
FROM team_ratings
WHERE gender = 'boys'
GROUP BY age_group
ORDER BY age_group;
```

Save this output. It is the rollback reference and the basis for post-change shift analysis.

---

### Section 4: Implementation Steps

ELO lists the specific steps in execution order:

1. **Confirm recompute scope:** Full recompute (all Boys ratings) or partial (specific age groups, specific leagues)?
2. **Apply calibration change:** Specific SQL UPDATE or config file change — ELO writes the exact command
3. **Run recompute:** Full or partial recompute command
4. **Verify recompute completion:** How does ELO confirm the recompute finished and results are written?

For each step, provide the exact command or SQL. Do not write "run the recompute" — write the actual recompute command.

---

### Section 5: Post-Execution Verification

After recompute completes, confirm Option A had the expected effect:

**Expected shifts (ELO pre-commits before running):**
- GA ASPIRE Boys teams: expected direction = [up/down], expected magnitude = [X–Y pts]
- MLS NEXT teams: expected direction = [up/down], expected magnitude = [X–Y pts]
- U13B / U14B age groups: expected shift = [X–Y pts]

**Verification queries:**

```sql
-- Distribution shift: Boys rating distribution post-Option-A
-- (Same as Section 3 snapshot — compare manually or via delta view)
SELECT 
    age_group,
    COUNT(*) as team_count,
    ROUND(AVG(rating), 1) as avg_rating,
    ROUND(PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY rating), 1) as p10,
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating), 1) as median,
    ROUND(PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY rating), 1) as p90
FROM team_ratings
WHERE gender = 'boys'
GROUP BY age_group
ORDER BY age_group;
```

```sql
-- MLS NEXT median check: expected 1400–1600 per Boys Option A query Q3
SELECT 
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating), 1) as mls_next_median
FROM team_ratings
WHERE gender = 'boys'
  AND [mls_next filter];
```

**Pass criteria:** If the distribution shift matches the expected direction and magnitude within ± 50 pts, ELO files the verdict as executed and notifies SENTINEL. If magnitude is outside ± 100 pts: escalate to SENTINEL before filing.

---

### Section 6: Post-Calibration Stability Monitoring

After Option A executes, ELO monitors for stability. Use the `Calibration Stability Baseline — April 29.md` queries adapted for the post-Option-A state:

- **Monitoring window:** 3 pipeline runs (approximately 3 days after pipeline is live)
- **Stability threshold:** Average Boys rating shift between pipeline runs < 5 pts per age group
- **If unstable (> 5 pts drift):** File SENTINEL flag with comparison table

---

### Section 7: Rollback Plan

If post-execution verification fails or the stability check shows unexpected drift:

1. Revert the calibration change (reverse Section 4 SQL)
2. Re-run recompute (same scope as Section 4)
3. Verify return to baseline using Section 3 snapshot values
4. File SENTINEL alert: "Boys Option A rolled back — post-execution verification failed at [specific check]"

**Rollback time estimate:** [ELO fills in: recompute time + verification time]

---

### Section 8: DSS Timing Note

If Boys Option A executes successfully and ELO files "calibration complete" before May 10, Boys calibration is in the DSS-ready state. If the verdict is delayed past May 5, Boys calibration may not have a full stability monitoring window before DSS (May 16). In that case, ELO flags in the DSS Calibration Evidence Summary: "Boys calibration (Option A) applied [date], monitoring window [X days] — stability confirmed/in-progress."

---

## Quality Standard

- Section 1 must contain specific parameter names, not category descriptions. If ELO does not know the exact parameter name, note "ELO will confirm from codebase on execution day" — but name the category so there is no ambiguity about what changes.
- Section 5 expected shifts must be filled in before the verdict arrives. ELO cannot write "the expected direction will be [depends on query results]" — the whole point of this protocol is that the expectations are pre-committed.
- Section 7 rollback time estimate must be specific. "Fast" is not an estimate.

## References

- `Rankings/Boys Option A — Decision Document.md`
- `Rankings/Boys Option A — Pre-Verdict Evidence Checklist.md`
- `Rankings/Boys Option A — Query Pack Validation Note.md`
- `Rankings/Calibration Stability Baseline — April 29.md`
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md`
- `Rankings/Boys Option A — DSS Calibration Evidence Summary.md`
