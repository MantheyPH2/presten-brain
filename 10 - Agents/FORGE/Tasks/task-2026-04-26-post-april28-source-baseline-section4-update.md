---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: urgent
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md (Section 4 updated)"
---

# Task: Post-April-28 Source Baseline — Section 4 Update

## Context

FORGE filed `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` with Section 4 containing placeholder verification checks for GA ASPIRE event_tier and ecnl_verified. Those checks were listed as pending April 28 execution. That execution will happen April 28.

FORGE's own briefing (2026-04-26 06:39) listed "Update Source Ingestion Baseline Section 4 (GA ASPIRE + ecnl_verified confirmation checks)" as a post-April-28 follow-up action. No task was created for it. This task closes that gap.

April 29 is the analysis session — the same session where ELO runs G1–G4 gates. FORGE must complete Section 4 before ELO's analysis concludes so the baseline document reflects confirmed post-fix state. The April 29 Decision Tree uses the baseline as a comparison anchor. If FORGE does not update Section 4, ELO is comparing against a stale pre-fix baseline.

## What to Produce

**Target file:** `02 - Tiger Tournaments/Projects/Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md`

Update Section 4 only. Do not modify other sections.

---

### Section 4: Post-April-28 Verification Checks (fill these in after April 28 runs)

**Check 1: GA ASPIRE Event Tier Confirmation**

```sql
-- Run after April 28 GA ASPIRE UPDATE and recompute
SELECT event_tier, COUNT(*) as game_count
FROM games
WHERE event_name ILIKE '%ga aspire%' OR event_tier = 'ga_aspire'
GROUP BY event_tier
ORDER BY game_count DESC;
```

Fill in:
| Field | Value |
|-------|-------|
| Run date/time | |
| ga_aspire rows found | |
| ga rows that should now be ga_aspire (were there any remaining?) | |
| Check result: PASS / FAIL | |

**PASS criteria:** All GA ASPIRE games have `event_tier = 'ga_aspire'`. Zero rows with `event_tier = 'ga'` where event name indicates GA ASPIRE.

**Check 2: ecnl_verified Column Existence**

```sql
-- Confirm column was added
SELECT column_name, data_type, column_default
FROM information_schema.columns
WHERE table_name = 'games' AND column_name = 'ecnl_verified';
```

Fill in:
| Field | Value |
|-------|-------|
| Run date/time | |
| Column present: YES / NO | |
| Data type matches spec (boolean) | |
| Default value confirmed (FALSE) | |
| Check result: PASS / FAIL | |

**PASS criteria:** Column exists, type is boolean, default is FALSE.

**Check 3: Baseline Game Count Post-Recompute**

```sql
SELECT COUNT(*) as total_games, MAX(updated_at) as last_updated
FROM games;
```

Fill in:
| Field | Value |
|-------|-------|
| Run date/time | |
| Total games post-recompute | |
| Delta from pre-April-28 baseline | |
| Last updated timestamp | |

Note any unexpected delta (large increase or decrease vs. Section 2 pre-fix total).

---

### Section 4 Summary Block

After filling all three checks, add a one-line status:

```
Section 4 Status: [All checks PASS / Check N FAIL — see notes]
Post-April-28 baseline confirmed: [YES / NO]
Filed by FORGE: [date/time]
```

---

## Quality Standard

- Pull actual query output from the April 29 psql session. Do not estimate or carry forward from prior briefings.
- If any check FAILS, flag in the next FORGE briefing immediately — do not wait until Section 4 is complete.
- The Section 4 Summary Block must be the last thing filled in, after all three checks pass.
- Do not reconstruct this from FORGE's memory of April 28 — use actual psql output.

## Sequence Dependency

This task depends on:
1. April 28 execution completing (GA ASPIRE UPDATE + ecnl_verified ALTER TABLE + recompute)
2. April 29 psql access

FORGE should run these checks at the start of the April 29 session, before ELO's G1–G4 analysis runs. The Section 4 confirmation feeds ELO's G1 gate (recompute confirmed complete).

## Why This Matters

The Source Ingestion Baseline is the pre-launch reference document SENTINEL and Presten use to understand what state the system was in before May 1. If Section 4 remains as placeholders, the document is incomplete and the May 1 first-week monitoring has no confirmed post-fix anchor to compare against. This is a 15-minute task if FORGE runs the three queries immediately on April 29.

## References

- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — file to update
- `Infrastructure/April 28 Reference Card.md` — the steps whose outputs Section 4 confirms
- `Infrastructure/April 28 Execution Log.md` — Presten's session record, cross-check Section 1–3 against it
- `Rankings/April 29 Post-Fix Decision Tree.md` — G1 gate uses Section 4 confirmation
