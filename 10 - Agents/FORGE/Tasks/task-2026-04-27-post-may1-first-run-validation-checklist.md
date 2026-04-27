---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: open
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 First-Run Validation Checklist.md"
---

# Task: May 1 First-Run Validation Checklist

## Context

FORGE filed the May 1 Deployment Day-Of Runbook (completed 2026-04-25) — a step-by-step execution document for running the pipeline on launch day. The runbook covers launch, monitoring, and a GREEN/YELLOW/RED outcome call.

What the runbook does not cover: **what to validate after the first run completes** to confirm the pipeline is actually working correctly — not just that it ran without crashing. A pipeline run can exit cleanly (GREEN) and still have silent failures: wrong org-IDs pulling incorrect teams, dedup logic misfiring, ELO updates skipping certain age groups, game counts far below baseline expectations.

This task: produce the validation checklist that FORGE runs on May 2 (the morning after the first May 1 run). This is the "did it actually work?" document, not the "did the process complete?" document.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 First-Run Validation Checklist.md`

This is a fill-in-the-blank execution document. FORGE opens it the morning of May 2 and works through each check.

---

### Pre-Checklist: Reference Numbers

Before running any checks, fill in the baseline numbers this validation compares against:

```
Pre-pipeline total games in DB (from Pre-April-28 Baseline Snapshot): ___________
Expected new games from May 1 run (estimate from Stage 1 org-ID scope): ___________
Expected org-IDs in Stage 1 crawl config: ___________
Expected pipeline runtime (from runbook): ___________
Actual May 1 run start time: ___________
Actual May 1 run end time: ___________
```

---

### Check 1: Game Count Delta

**Query to run:**
```sql
SELECT COUNT(*) FROM games WHERE created_at >= '[May 1 pipeline run timestamp]';
```

| Metric | Expected | Actual | Status |
|--------|----------|--------|--------|
| New games ingested | [N from estimate] | | PASS / FLAG |
| New games as % of baseline | [N%] | | PASS / FLAG |

**PASS criteria:** Actual new games ≥ 50% of expected estimate.
**FLAG criteria:** Actual new games < 50% of expected, OR 0 new games despite GREEN pipeline exit.
**If FLAG:** Note specific org-IDs that may have returned empty and escalate to SENTINEL.

---

### Check 2: Org-ID Coverage Verification

Verify each Stage 1 org-ID actually contributed games:

| Org-ID | Entity | Games Ingested | Status |
|--------|--------|---------------|--------|
| [org-id-1] | [entity] | | PASS / ZERO |
| [org-id-2] | [entity] | | PASS / ZERO |
| ... | | | |

Fill this table from the Stage 1 org-ID config. Any org-ID showing ZERO games is a potential config error, auth issue, or the entity simply has no recent games — flag for review.

**PASS criteria:** ≥ 70% of org-IDs contributed at least 1 game.
**FLAG criteria:** Multiple org-IDs returning ZERO when they should have games for this period.

---

### Check 3: Team Identity Integrity

Spot-check that newly ingested games are assigned to the correct team records (not creating phantom teams or misassigning to wrong teams):

Pick 5 games at random from the May 1 ingest batch. For each:

| Game ID | Home Team (DB) | Away Team (DB) | Team IDs Look Correct? | Notes |
|---------|----------------|----------------|------------------------|-------|
| | | | YES / FLAG | |
| | | | YES / FLAG | |
| | | | YES / FLAG | |
| | | | YES / FLAG | |
| | | | YES / FLAG | |

"Looks correct" = team names match expected age group and gender, no obvious mismatches like a U16 Boys team appearing in a U13 Girls game record.

**FLAG criteria:** Any game where team identity appears wrong — name mismatch, age/gender inconsistency.

---

### Check 4: ELO Update Confirmation

Confirm ELO recompute ran for the newly ingested games:

```sql
SELECT COUNT(DISTINCT team_id) FROM elo_history 
WHERE updated_at >= '[May 1 pipeline run timestamp]';
```

| Metric | Expected | Actual | Status |
|--------|----------|--------|--------|
| Teams with updated ELO ratings | [> 0] | | PASS / FAIL |

**PASS criteria:** At least 1 team ELO updated (if any games were ingested).
**FAIL criteria:** 0 teams updated despite games being ingested — ELO step may have silently failed.

---

### Check 5: DLQ Review

Check the Dead Letter Queue for May 1 processing failures:

```
DLQ entry count from May 1 run: ___________
```

| DLQ Count | Interpretation | Action |
|-----------|---------------|--------|
| 0 | Clean run | No action |
| 1–5 | Expected noise | Log, no escalation |
| 6–15 | Elevated — review entries | File DLQ entries in briefing |
| 16+ | Abnormal — classify each | SENTINEL flag required |

List all DLQ entries if count is ≥ 6, using the error taxonomy from `Infrastructure/Pipeline Error Classification Taxonomy.md`.

---

### Check 6: Archive State Confirmation (if archive ran before May 1)

If the archive dry-run ran and Presten authorized the production pass before May 1:

| Metric | Expected | Actual | Status |
|--------|----------|--------|--------|
| De-prioritized teams excluded from run | ~155–165 | | PASS / FLAG |
| Active team count in this run | [N total minus archived] | | |

If archive did NOT run before May 1: note this and confirm all 180 de-prioritized teams ran through the pipeline. They will generate expected noise (empty crawl results). Do not treat this as an error.

---

### Final Verdict

```
Checks 1–5 all PASS:         [ ] → GREEN — pipeline running correctly
Any Check 2–5 FAIL:          [ ] → YELLOW — specific issue identified, escalate to SENTINEL
Check 1 at <10% of expected: [ ] → RED — pipeline ran but produced near-nothing, investigate before May 2 run
```

---

## Quality Standard

- The reference numbers in the pre-checklist must be filled in before running any checks — not estimated after the fact.
- Every FLAG entry must be described in the following FORGE briefing with the specific org-ID, game ID, or team ID that failed.
- The Final Verdict must be one of the three options. Do not leave it blank or open-ended.
- This document should take ≤ 30 minutes to complete on May 2 morning.

## Why This Matters

The May 1 Deployment Runbook tells FORGE how to run the pipeline. This checklist tells FORGE how to confirm it worked. A pipeline that exits GREEN but produces 0 games is a silent failure — the runbook won't catch it, but Check 1 will. A pipeline that misassigns team IDs will corrupt rankings — Check 3 will catch it. SENTINEL needs the May 2 validation result to know whether the May 1 launch succeeded in substance, not just in process. Without this checklist, SENTINEL's post-launch briefing review is based on "the runbook showed GREEN" rather than "games ingested, coverage verified, ELO updated."

## References

- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — the launch execution this validates
- `Infrastructure/Pipeline Error Classification Taxonomy.md` — error codes for DLQ classification
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — baseline game counts
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — Stage 1 org-ID list for Check 2
