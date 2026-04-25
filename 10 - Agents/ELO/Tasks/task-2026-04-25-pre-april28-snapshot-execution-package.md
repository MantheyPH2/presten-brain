---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: urgent
due: 2026-04-26
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md"
---

# Task: Pre-April-28 Baseline Snapshot — Presten Execution Package

## Context

The Pre-April-28 Baseline Snapshot Spec (`Rankings/Pre-April-28 Baseline Snapshot Spec.md`) defines three queries Presten must run before the April 28 GA ASPIRE fix session. Without the snapshot, ELO can confirm the direction of the fix on April 29 but not the magnitude. Without magnitude, the rating shift analysis cannot answer whether the fix was proportionate or overcorrected.

The spec exists. The problem: it is written as a planning document — it explains what to capture and why, not how to run it. Presten needs a ready-to-open execution package with copy-paste queries, exact output format, and instructions for saving the three output files.

ELO's own April 25 00:56 briefing lists this as PRIORITY today or April 27 latest. This task formalizes the assignment.

## What to Do

### Step 1: Read the Spec

Read `02 - Tiger Tournaments/Projects/Rankings/Pre-April-28 Baseline Snapshot Spec.md` in full. Extract the three queries and the three output files they produce.

### Step 2: Convert to Execution Format

For each of the three queries:

1. Write the exact SQL — copy-paste ready, zero placeholders
2. Specify the exact filename for saving output (e.g., `pre-april28-snapshot-ratings.csv` — match spec)
3. Write one line on how to save: `\COPY (query) TO 'filename.csv' WITH CSV HEADER` or equivalent for the psql access method Presten uses
4. Write the expected output: how many rows to expect, what the column headers should be

Do not add explanatory prose around the queries. Presten does not need to understand why during execution — the spec covers that.

### Step 3: Add Pre-Run Gate and Post-Run Confirmation

Before the three queries, add a pre-run gate:

```
Pre-Run Gate:
[ ] Today is April 26 or April 27
[ ] April 28 session has NOT yet run (fix has not been applied)
[ ] psql access to youth_soccer DB is confirmed
→ If all three checked: proceed. If any unchecked: stop.
```

After the three queries, add a post-run confirmation:

```
Post-Run Confirmation:
[ ] File 1 saved: [filename] — [expected row count] rows
[ ] File 2 saved: [filename] — [expected row count] rows
[ ] File 3 saved: [filename] — [expected row count] rows
→ All three files saved: baseline snapshot complete. April 28 session can proceed.
→ Missing files: run the missing query before April 28.
```

### Step 4: Write the Delivery Document

Deliverable: `02 - Tiger Tournaments/Projects/Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md`

Total length: target under 1 page. This is an execution document, not a reference doc. Speed and clarity over completeness.

Sections:
1. **When to Run** (one sentence: April 26 or April 27, before April 28 session)
2. **Pre-Run Gate** (checkbox block)
3. **Query 1** (SQL + save command + expected output)
4. **Query 2** (SQL + save command + expected output)
5. **Query 3** (SQL + save command + expected output)
6. **Post-Run Confirmation** (checkbox block)
7. **Done** (one line: "Baseline captured. ELO uses these files for April 29 rating shift analysis.")

## Definition of Done

- All three queries are copy-paste ready
- All three output filenames match the spec exactly
- Pre-run gate and post-run confirmation are checkbox format
- Document is under 1 page
- No `?` or TBD in any query — if any column names are uncertain, resolve them against the vault schema references before filing
- Summary in briefing: "Pre-April-28 Execution Package filed. 3 queries, 3 output files. Pre-run gate and post-run confirmation included. Presten can complete in under 10 minutes."

## Why This Matters

April 28 is 3 days away. The baseline snapshot must run April 26 or 27 — today is the last reliable session before that window. If the snapshot doesn't run before April 28, ELO's April 29 analysis is directional only. SENTINEL is treating this as urgent because a directional-only analysis is insufficient to authorize Event Strength Phase 1 to run.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Pre-April-28 Baseline Snapshot Spec.md`
- `02 - Tiger Tournaments/Projects/Rankings/Post-April-28 Rating Shift Analysis Spec.md`
- `02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md`
