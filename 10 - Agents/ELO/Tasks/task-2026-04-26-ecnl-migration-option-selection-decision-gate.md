---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — Option Selection Decision Gate.md"
---

# Task: ECNL Migration — Option Selection Decision Gate

## Context

The ECNL Migration Rating Continuity Spec (filed April 25) documents multiple options for how to handle ECNL team ratings when the June 1 migration occurs (the transition from treating ECNL games as one league to the new structure). The spec defines the options and their trade-offs. It does not define when or how ELO decides which option to use.

Between now and June 1 there are several data points that will inform the option selection:
- April 29: ecnl_verified backfill confirmed (or not) — affects Option 2 validity
- May 1: Daily pipeline launches; ECNL forward ingestion accuracy confirmed (or not)
- May 1: ECNL RL staleness verification runs (execution package ready)
- May 5–9: DSS readiness assessment; ECNL coverage is a background assumption
- May 10: ELO's self-imposed deadline for ECNL migration option selection

Without a decision gate document, ELO cannot track whether the evidence needed for option selection is accumulating. The option selection may land on May 10 with incomplete data because no one tracked what was still missing.

This task: ELO writes the decision gate now. It defines what evidence maps to which option, what data must be collected by what date, and what SENTINEL authorizes. By May 5, ELO updates this document with all collected evidence. By May 10, ELO files the final option selection.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — Option Selection Decision Gate.md`

---

### Header

```
Status: IN PROGRESS — collecting evidence
Final selection due: 2026-05-10
June 1 migration date: 2026-06-01
Days until migration at time of writing: 36
```

---

### Section 1: Options Summary (reference from Rating Continuity Spec)

ELO summarizes the options from `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` in one table — option letter, name, one-sentence description, key assumption. Do not reproduce the full spec; this is a reference summary.

| Option | Name | One-Line Description | Key Assumption |
|--------|------|---------------------|----------------|
| A | | | |
| B | | | |
| C | | | |

(Fill from the Continuity Spec — do not invent options.)

---

### Section 2: Evidence Required Per Option

For each option, what evidence does ELO need to confirm the option is viable?

| Option | Required Evidence | Status | Source | Due Date |
|--------|------------------|--------|--------|----------|
| A | | | | |
| B | | | | |
| C | | | | |

ELO fills this section from the Rating Continuity Spec's assumptions. If Option A assumes `ecnl_verified` is accurate for historical games, then the required evidence is "ecnl_verified backfill confirmed complete (April 28 + ecnl_verified audit)."

**Status values:** COLLECTED / PENDING / BLOCKED

---

### Section 3: Disqualifying Conditions

If any of these conditions are true, the corresponding option is disqualified regardless of other evidence:

| Condition | Disqualifies |
|-----------|-------------|
| ECNL RL staleness check returns FAIL (> 30 days stale) | Any option that relies on ECNL RL as a reliable source |
| ecnl_verified forward ingestion not fixed before May 1 | Any option that relies on ecnl_verified accuracy for post-April-28 games |
| FM2 returns Path C (structural scraper gap) and not resolved before June 1 | Options that treat ecnl_verified as a reliable forward signal |
| April 29 G2 returns FAIL (Girls calibration unstable) | Any option that applies ECNL migration on top of unstable calibration |

ELO adds any additional disqualifying conditions identified during the spec review.

---

### Section 4: Evidence Collection Timeline

| Date | Data Point | Who Collects | Status |
|------|------------|-------------|--------|
| April 29 | ecnl_verified backfill confirmed complete | Presten (April 28 Execution Log) | PENDING |
| April 29 – May 1 | ECNL RL staleness verification queries | Presten (Execution Package filed) | PENDING |
| May 1 | Daily pipeline launches; first ECNL game ingestion verified | FORGE (post-launch monitoring) | PENDING |
| May 3 | USA Rank comparison run (girls, all age groups) | Presten + ELO | PENDING |
| May 5 | ecnl_verified forward accuracy confirmed (one week of pipeline data) | FORGE weekly health review | PENDING |
| May 8 | First weekly pipeline health review — ECNL source freshness | FORGE | PENDING |
| May 10 | **Option selection due — ELO files final choice** | ELO | PENDING |

ELO updates this table as each data point is collected.

---

### Section 5: Option Selection Criteria

After all evidence is collected (by May 5), ELO applies this decision logic:

1. Disqualify any options with unmet disqualifying conditions (Section 3)
2. Among remaining options, select the option with the most collected evidence (Section 2)
3. If two options are tied: select the lower-risk option (prefer preserving historical rating continuity over maximizing new-data accuracy)

ELO documents the selection here:

```
Option selected: ___________
Date selected: ___________
Reason (one paragraph): ___________
Disqualified options (if any) and reason: ___________
Open risks with selected option: ___________
```

---

### Section 6: SENTINEL Authorization

> SENTINEL fills — do not pre-fill.

```
Option selection reviewed: [ ]
Evidence reviewed: [ ]
Option [letter] authorized: YES / NO
If NO: ___________
SENTINEL sign-off: ___________
Review date: ___________
```

---

## Quality Standard

- Section 2 evidence table must reference specific existing documents (not vague references like "verify data quality")
- Section 3 disqualifying conditions must be concrete and binary — no "may affect" language
- Section 4 timeline must be complete by April 29 (first data point imminent)
- Section 5 decision criteria must be deterministic: given the evidence table, an observer other than ELO should reach the same option selection
- Final selection (Section 5) filed by May 10

## Why This Matters

June 1 is 36 days away. Option selection by May 10 gives 3 weeks for SENTINEL review and any implementation prep. If option selection slips to May 15 or later, there is no buffer for SENTINEL questions, Presten approval, or unexpected complications. This document ensures ELO tracks evidence collection across 5 data points over 10 days and arrives at May 10 with a decision — not with an open question.

## References

- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` — options source
- `Infrastructure/ECNL RL Staleness Verification — Presten Execution Package.md` — April 29–May 1 data point
- `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md` — FM2 disqualifying condition source
- `Rankings/April 29 Decision Tree — Verdict Filing Document.md` — G4 and calibration inputs
- `Rankings/ECNL Migration Pre-Flight Checklist — June 2026.md` — downstream dependency
