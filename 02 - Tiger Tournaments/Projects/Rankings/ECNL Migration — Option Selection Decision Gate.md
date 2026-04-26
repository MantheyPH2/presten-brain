---
title: ECNL Migration — Option Selection Decision Gate
type: decision-gate
author: ELO
date: 2026-04-26
status: in-progress
tags: [ecnl, migration, decision-gate, rating-continuity, june-1, evo-draw]
related: "[[ECNL Migration Rating Continuity Spec — June 2026]]", "[[ECNL Rating Continuity Spec — June 2026]]", "[[April 29 Decision Tree — Verdict Filing Document]]"
---

# ECNL Migration — Option Selection Decision Gate

> **What this document is:** Evidence tracker and decision framework for selecting the ECNL tier handling approach before June 1 migration. ELO updates this document as evidence accumulates. Final option selection filed by May 10. SENTINEL authorizes in Section 6.

---

## Header

```
Status: IN PROGRESS — collecting evidence
Final selection due: 2026-05-10
June 1 migration date: 2026-06-01
Days until migration at time of writing: 36
Last updated: 2026-04-26
```

---

## Section 1: Options Summary

From `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md`. Read that spec for full analysis before making selection.

| Option | Name | One-Line Description | Key Assumption |
|--------|------|---------------------|----------------|
| 1 | No Re-tag (Recommended) | Keep historical `ecnl` games tagged as `ecnl`; new post-June-1 games tagged `ecnl_rl`; no recompute | Historical ECNL games legitimately reflect higher-calibration competition; natural recency weighting compresses Path A vs Path B gap over 6–12 months |
| 2 | Re-tag to ecnl_rl | Update all historical `ecnl` games for Path A teams to `ecnl_rl` on June 1; run full recompute | Historical `ecnl` calibration inflates ratings unfairly vs. teams always in ECNL RL; clean convergence on June 1 is worth the rating cliff |
| 3 | ecnl_legacy tier | Introduce new `event_tier = 'ecnl_legacy'` for pre-migration games with a blended calibration value | A purpose-built transition tier produces smoother rating continuity than either preserving `ecnl` cal or re-tagging to `ecnl_rl` cal |

**ELO current lean:** Option 1. Reasoning is in the full Continuity Spec Section 3. Option 2 creates a 40–80 pt rating cliff for Path A teams on June 1. Option 3 requires DDL changes and a new calibration value ELO has no cross-league data to support.

---

## Section 2: Evidence Required Per Option

| Option | Required Evidence | Status | Source | Due Date |
|--------|-----------------|--------|--------|----------|
| 1 | ecnl_verified backfill confirmed complete (Path A teams identifiable) | PENDING | April 28 Execution Log Step 2b | April 29 |
| 1 | ECNL RL staleness check passes (≤ 30 days stale) — confirms ECNL RL is live data source | PENDING | ECNL RL Staleness Verification Execution Package | April 29–May 1 |
| 1 | April 29 G2 PASS (Girls calibration stable — not running migration on top of instability) | PENDING | April 29 Verdict Filing Document Gate G2 | April 29 |
| 2 | Path A team list confirmed and rated (need count of teams × avg ECNL game volume to model cliff magnitude) | PENDING | Path A confirmation query (Continuity Spec §1) | May 5 |
| 2 | Presten sign-off on Option 2 rating cliff (required per Continuity Spec §5 — affects 1,000+ team ratings) | PENDING | SENTINEL brief to Presten | Before May 10 |
| 3 | Cross-league calibration data supporting a blended `ecnl_legacy` cal value | BLOCKED | No current data source — not derivable from existing analysis | N/A |
| 3 | DDL change authorized by SENTINEL | PENDING | SENTINEL authorization | Before May 10 |

---

## Section 3: Disqualifying Conditions

If any condition in this table is true, the corresponding option is disqualified. Check each condition before filing the May 10 option selection.

| Condition | Disqualifies | Check Date |
|-----------|-------------|------------|
| ECNL RL staleness check returns FAIL (> 30 days stale — ECNL RL is not a reliable source) | Option 1 (rating continuity depends on new ECNL RL games arriving correctly) | April 29–May 1 |
| ecnl_verified forward ingestion not confirmed accurate by May 1 (FM2 unresolved) | Option 1 if it relies on ecnl_verified accuracy for forward Path A identification | May 1 |
| April 29 G2 FAIL (Girls calibration still unstable) | All options — do not apply migration tier logic on top of unstable calibration; hold entire ECNL migration plan until G2 passes | April 29 |
| Path A team count > 1,000 AND Presten has not approved Option 2 rating cliff | Option 2 — Continuity Spec §5 requires Presten sign-off for changes affecting 1,000+ ratings | May 5 |
| No cross-league calibration data available to derive `ecnl_legacy` cal value by May 5 | Option 3 — cannot set a calibration value without data | May 5 |

---

## Section 4: Evidence Collection Timeline

ELO updates the Status column as each data point is collected.

| Date | Data Point | Who Collects | Status |
|------|-----------|-------------|--------|
| April 29 | ecnl_verified backfill confirmed complete (April 28 Execution Log Step 2b) | Presten (April 28 session record) | PENDING |
| April 29 | April 29 G2 PASS confirmed (Girls calibration post-fix stable) | ELO (Verdict Filing Document) | PENDING |
| April 29–May 1 | ECNL RL staleness verification queries | Presten (ECNL RL Staleness Execution Package) | PENDING |
| May 1 | Daily pipeline launches; first ECNL game ingestion verified | FORGE (post-launch monitoring) | PENDING |
| May 5 | ecnl_verified forward accuracy confirmed (one week of pipeline data) | FORGE weekly health review | PENDING |
| May 5 | Path A team count and avg ECNL game volume (if Option 2 still viable) | ELO — run Continuity Spec §1 confirmation query | PENDING |
| May 8 | First weekly pipeline health review — ECNL source freshness (ECNL RL data current) | FORGE | PENDING |
| **May 10** | **Option selection due — ELO files final choice (Section 5)** | **ELO** | **PENDING** |

---

## Section 5: Option Selection Criteria and Decision

After all evidence is collected (target: May 5), ELO applies this logic in order:

1. **Disqualify** any options with unmet disqualifying conditions (Section 3)
2. **Among remaining options**, select the option with the most collected evidence (Section 2 Status = COLLECTED)
3. **Tiebreaker:** If two options are equally evidenced, prefer lower implementation risk. Specifically:
   - Option 1 < Option 2 < Option 3 in implementation cost and reversibility
   - Option 1 requires no SQL writes; Option 2 requires a mass UPDATE + recompute; Option 3 requires DDL
4. **If Option 1 remains viable after all evidence is collected, ELO selects Option 1** — this is the current recommendation and requires no schema changes.

**Final selection (ELO fills May 10):**

```
Option selected: ___________
Date selected: ___________

Reason (one paragraph):
___________

Disqualified options (if any) and reason:
___________

Open risks with selected option:
___________
```

---

## Section 6: SENTINEL Authorization

> **SENTINEL fills — do not pre-fill.**

```
Option selection reviewed: [ ]
Evidence reviewed (Section 2 status check): [ ]
Option [letter] authorized: YES / NO

If NO — reason and next steps:
___________

SENTINEL sign-off: ___________
Review date: ___________
```

---

## Downstream Dependencies

This option selection gates the following:
- **FORGE:** FORGE must confirm event classifier is updated for ECNL RL events before June 1 — the mechanism differs slightly by option
- **Pre-June-1 migration prep:** Under Option 1, no additional DB writes needed before June 1 beyond normal `team_merges` insertion (per ECNL Rating Continuity Spec)
- **June 2 verification:** The post-migration verification query (Continuity Spec §4) must be run before recompute on June 2, regardless of option

---

## References

- [[ECNL Migration Rating Continuity Spec — June 2026]] — full options analysis; Section 3 has ELO's reasoning for Option 1
- [[ECNL Rating Continuity Spec — June 2026]] — verified-flag mechanism; prerequisite for all options
- [[April 29 Decision Tree — Verdict Filing Document]] — G2 and G4 gates feed Section 4 evidence collection
- [[June 1 Risk Assessment — Age Group Transition + ECNL Migration]] — full migration risk context
- `Infrastructure/ECNL RL Staleness Verification — Presten Execution Package.md` — Section 4 April 29–May 1 data point
- `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md` — FM2 disqualifying condition source

*ELO — 2026-04-26*
