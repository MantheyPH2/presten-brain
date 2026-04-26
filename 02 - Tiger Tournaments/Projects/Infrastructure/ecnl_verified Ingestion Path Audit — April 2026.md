---
title: ecnl_verified Ingestion Path Audit — April 2026
tags: [infrastructure, pipeline, ecnl, ecnl-verified, audit, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: outcome-c — code access required
due: 2026-04-29
related: "[[Step 2b — ecnl_verified Backfill SQL Execution Package]]", "[[ECNL ecnl_verified Column — Population Plan]]", "[[Pipeline Code Review Results]]"
---

# ecnl_verified Ingestion Path Audit — April 2026

> **Audit Verdict: OUTCOME C — Pipeline write path not audited. Presten must confirm ecnl_verified assignment before May 1 deployment. See Section 3.**

---

## Purpose

The Step 2b backfill (`UPDATE games SET ecnl_verified = TRUE WHERE source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'`) fixes all ECNL games currently in the database. That runs April 28.

Starting May 1, the daily pipeline ingests new ECNL games every day. This audit answers: when a new ECNL game is written to the `games` table via the pipeline, does it receive `ecnl_verified = TRUE` automatically?

If not: every new ECNL game after April 28 will have `ecnl_verified = FALSE` (the column default). The historical backfill is complete, but the column is permanently inaccurate for all future data.

---

## Section 1: Pipeline Write Path for ECNL Games

**FORGE Finding:** The pipeline codebase is not stored in the Obsidian vault. FORGE cannot directly inspect the INSERT/upsert logic.

Based on vault documentation:
- ECNL games enter via the **TGS scraper** (`source = 'tgs'`) and the **TGS AthleteOne scraper** (`source = 'tgs_athleteone'`)
- The write path is expected to be in the TGS scraper module, where game records are assembled and written to the `games` table
- The `ecnl_verified` column was **newly added** as part of the April 28 session (ALTER TABLE). Before April 28, it did not exist — so the scrapers were definitely NOT setting it before that date.

**The core question:** Has the TGS scraper code been updated (per `Infrastructure/ECNL ecnl_verified Column — Population Plan.md`) to set `ecnl_verified = TRUE` for qualifying games at write time? This code change was planned but may not yet be implemented.

**Required action for Presten:** This is FM2 in `Infrastructure/Pipeline Code Review Results.md` (FM2-1 through FM2-3).

---

## Section 2: ecnl_verified Assignment Logic

**What Presten must check:**

1. Search codebase for `ecnl_verified` in the TGS scraper.
   - **If found:** Note the file and line. Confirm the assignment condition matches the backfill: `source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'`. If the scraper uses `ecnl_verified: true` in the game INSERT object, confirm this is only set when the event is confirmed ECNL (not for ECNL RL or other tiers).
   - **If not found:** The column is not yet written by the scraper. All new ECNL games from May 1 forward will default to `ecnl_verified = FALSE`.

2. If `ecnl_verified` is set at write time, check whether the assignment condition is consistent with the backfill:
   - Backfill condition: `source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'`
   - If scraper uses different logic (e.g., checks a different field, or sets TRUE for all TGS games regardless of tier), note the discrepancy — it may over- or under-flag games.

3. Confirm whether the `ECNL ecnl_verified Column — Population Plan` code change has already been applied to the TGS scraper. The plan called for a code change to the scraper before May 1. If applied: FM2 Path A. If not applied: FM2 Path B.

---

## Section 3: Audit Verdict

### OUTCOME C — Code Not Accessible

FORGE cannot access the TGS scraper codebase from the vault. The `ecnl_verified` assignment logic was not confirmed.

**SENTINEL must request Presten confirm `ecnl_verified` assignment before May 1 deployment.**

The mechanism: `Infrastructure/Pipeline Code Review Results.md` — FM2 checks. Presten should complete FM2-1 through FM2-3 and return results to FORGE before April 29.

**If FM2 Path = A (scraper already sets ecnl_verified correctly):** No code change needed. May 1 deployment proceeds without additional conditions on this item.

**If FM2 Path = B (scraper does not set ecnl_verified, simple addition):** FORGE recommends the following minimum code change in the TGS scraper game INSERT/upsert:

```javascript
// In TGS scraper game write logic — add to INSERT payload
ecnl_verified: (
  ['tgs', 'tgs_athleteone'].includes(source) &&
  event_tier === 'ecnl'
),
```

This is a ≤ 5-line change. **It must be deployed with or before the May 1 pipeline launch.** If deployed after May 1, all ECNL games ingested between May 1 and the fix date will have `ecnl_verified = FALSE` and will require a supplemental backfill — against a larger dataset than the April 28 backfill.

Add this condition to the **May 1 Deployment Pre-Authorization Checklist**, Section 3 (run-daily.sh section), as an additional pre-flight check: "ecnl_verified scraper code change confirmed applied: YES / NO."

**If FM2 Path = C (scraper architecture prevents simple field addition):** FORGE escalates to SENTINEL. Stage 2 authorization is held until the `ecnl_verified` fix is confirmed deployed.

---

## Section 4: Interaction with ECNL Migration

The `ECNL Migration Rating Continuity Spec — June 2026.md` (ELO) depends on `ecnl_verified` accurately flagging historical ECNL games for the migration rating continuity model.

If this audit returns Outcome B or C:
- All ECNL games ingested after April 28 will have `ecnl_verified = FALSE`
- The distinction between "verified historical ECNL" (pre-April-28, backfilled) and "unverified new ECNL" (post-April-28, no scraper fix) becomes ambiguous
- ELO's Option 2 in the migration spec (re-tag historical games using ecnl_verified) loses reliability if the column's accuracy degrades with each new ingestion cycle
- **ELO should be notified of the Outcome B/C finding** so the migration spec can note this dependency explicitly

This is not an emergency for April 28–May 1, but it will become one by June 1 migration planning if not resolved.

---

## Impact Timeline

| Timeline | Impact if FM2 = B and not fixed before May 1 |
|----------|----------------------------------------------|
| May 1–7 | New ECNL games accumulate with `ecnl_verified = FALSE`. Dedup logic that depends on this field begins drifting. Low volume early in the week — small absolute impact. |
| May 8 (first weekly health review) | Health review query for `ecnl_verified` coverage will show new ECNL games appearing as unverified. The issue becomes visible. |
| May 9 (DSS) | If demo uses ECNL migration readiness as a data quality claim, `ecnl_verified` accuracy is part of that claim. Partially inaccurate column weakens the claim. |
| June 1 (ECNL migration) | ECNL migration logic built on `ecnl_verified` has a growing body of silently excluded games. ELO must re-run backfill before migration can proceed. |

---

## References

- `Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md` — one-time backfill this audit complements
- `Infrastructure/ECNL ecnl_verified Column — Population Plan.md` — scraper code change plan
- `Infrastructure/Pipeline Code Review Results.md` — FM2 checklist (Presten fills)
- `Infrastructure/April 28 Execution Log.md` — Step 2b is logged here; forward-looking gap closed by this audit
- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — add FM2 condition if Path = B
- `task-2026-04-26-ga-aspire-pipeline-classifier-audit.md` — parallel audit for GA ASPIRE classification
- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` — downstream dependency on ecnl_verified accuracy

*FORGE — 2026-04-26*
