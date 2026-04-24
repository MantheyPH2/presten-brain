---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: completed
priority: p0
due: 2026-04-28
completed: 2026-04-24
deliverable: Reports/ecnl-dedup-verified-fix-2026-04-24.md
---

# Task: Fix `ecnl-transition-dedup.js` — Add `verified = true` to Merge INSERT

## Context

ELO's June 1 Risk Assessment identified a P0 defect in the ECNL migration pipeline. When `ecnl-transition-dedup.js` inserts records into `team_merges` to map TGS team IDs → GotSport team IDs on June 1, it must include `verified = true` in the INSERT. The Ranking Engine (Phase 1) only loads merges where `verified = true`. Without this flag, every ECNL team whose platform ID changes on June 1 silently loses its entire game history and resets to ~1000 rating overnight.

Estimated scope: 500–2,000 ECNL teams. This is a five-minute code fix. It is not assigned to anyone yet. June 1 is 38 days out.

## What You Need to Do

### Step 1: Locate the Script and Confirm the Bug

Find `ecnl-transition-dedup.js` in the codebase (likely in `scripts/` or the pipeline root). Read the full script.

Find the `INSERT INTO team_merges` statement(s). Check whether `verified` is included in the column list and whether it is set to `true`.

Document exactly:
- File path (relative to project root)
- Line number(s) of the INSERT statement
- Current column list in the INSERT
- Current value for `verified` (present as `false`, present as `null`, or missing entirely)

### Step 2: Write the Fix

If `verified` is not set to `true`, the fix is one of:

**Case A — `verified` column is present but set to `false` or `null`:**
Change the value to `true`.

**Case B — `verified` column is absent from the INSERT:**
Add it to both the column list and values list:
```js
// Before (example — adapt to actual code):
await db.query(`
  INSERT INTO team_merges (primary_team_id, secondary_team_id, method, created_at)
  VALUES ($1, $2, $3, NOW())
`, [primaryId, secondaryId, 'ecnl-migration']);

// After:
await db.query(`
  INSERT INTO team_merges (primary_team_id, secondary_team_id, method, verified, created_at)
  VALUES ($1, $2, $3, true, NOW())
`, [primaryId, secondaryId, 'ecnl-migration']);
```

Adapt to the actual code structure — do not copy this verbatim if the schema or query style differs.

### Step 3: Check for All INSERT Sites

Search the file for every `INSERT INTO team_merges` occurrence. The fix must apply to all of them, not just the first one found. If the script has a batch insert path and a single-record insert path, both must include `verified = true`.

Also check whether any test files for this script mock the INSERT — if so, note the test file paths but do not modify test files (that is Presten's call).

### Step 4: Verify Against the Schema

Confirm the `team_merges` table has a `verified` column by referencing `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md`. If the schema doc doesn't confirm it, note that a DB schema check is needed before deployment:
```sql
SELECT column_name, data_type, column_default
FROM information_schema.columns
WHERE table_name = 'team_merges' AND column_name = 'verified';
```

### Step 5: Write the Deliverable

Write a concise fix note to: `Reports/ecnl-dedup-verified-fix-2026-04-24.md`

Include:
1. File path and line number of the fix
2. Before/after code diff (exact lines)
3. Confirmation that all INSERT sites are covered
4. Schema verification status (confirmed from docs, or query needed)
5. Whether any test files reference this INSERT and need review

## Definition of Done

- Bug confirmed: `verified` is missing or false in the current INSERT
- Fix written as an exact code diff (not pseudocode)
- All INSERT sites in the file are covered
- Deliverable note filed at `Reports/ecnl-dedup-verified-fix-2026-04-24.md`
- Summary in your next briefing: file path, line number, what changed

## If the Bug Is Not Present

If `verified = true` is already set correctly in all INSERT sites, document this in `Reports/ecnl-dedup-verified-fix-2026-04-24.md` as a confirmed-safe finding. Include the exact line(s) showing the correct value. This closes ELO's P0 flag with evidence.

## Why This Cannot Wait

ELO's risk assessment estimates a partial mitigation path (pre-merge TGS→GotSport IDs before June 1) but that mitigation depends on this code being correct when the pre-merge runs. If the code runs on June 1 with `verified` absent, all the pre-migration work is wasted — the Ranking Engine will ignore the new merge records entirely.

This is the smallest task on the board with the largest downside if missed.

## References

- `02 - Tiger Tournaments/Projects/Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md` — ELO's analysis (Section: Risk B, ECNL ID Discontinuity)
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — `team_merges` table definition
- SENTINEL Briefing 2026-04-23 23:52 — P0 alert section
