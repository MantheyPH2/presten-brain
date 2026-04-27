---
title: FM2 Active — Code Change Specification
tags: [infrastructure, pipeline, ecnl, ecnl-verified, fm2, code-change, forge]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: pre-written — activate if FM2 path = B or C confirmed by Presten
task: task-2026-04-27-fm2-active-code-change-spec
---

# FM2 Active — Code Change Specification

> **How to use:** This document is a pre-written spec for the FM2 code path. It is not activated unless Presten files FM2 Path B or C in `Infrastructure/Pipeline Code Review Results.md`. If FM2 = Path A (already writes `ecnl_verified` correctly), close this document — no action needed.

---

## Section 1: FM2 Code Path Description

### What FM2 Does

FM2 is the code path check for whether the TGS scraper (the ECNL game ingestion path) writes the `ecnl_verified` field to the database at insert time.

- **Which file(s):** Based on the FM2 Quick-Run Guide, the target is the TGS scraper game INSERT/upsert — expected at a path similar to `crawlers/tgs/ingest.js`, `scrapers/tgs/writer.js`, or `lib/tgs/game-writer.js`. Presten's FM2-1 grep result (`ecnl_verified` in TGS scraper files) is the authoritative file:line reference. If FM2-1 = "not found," Section 2 below applies directly.

- **What FM2 does that FM1 does not:** FM1 controls the `event_tier` classification for the GotSport scraper (`ga_aspire` vs. `ga`). FM2 is an independent concern: it controls whether ECNL games ingested via TGS receive `ecnl_verified = TRUE` at write time. These are separate scrapers, separate fields, and separate code paths.

- **Consequence if FM2 runs with old behavior on April 28 recompute:**
  - The April 28 session adds the `ecnl_verified` column with default `FALSE`.
  - If FM2 is not writing the field, every new ECNL game ingested after May 1 will land with `ecnl_verified = FALSE`.
  - The April 29 backfill (`ecnl_verified — Backfill SQL Package.md`) fixes existing records but does not affect forward ingestion.
  - ELO's CP2 checkpoint queries `ecnl_verified` population rate for ECNL games. Post-May-1 ECNL games with `ecnl_verified = FALSE` will appear as un-verified in the CP2 count, producing a false-low population rate as the dataset grows.
  - By June 2026 (ECNL migration decision window), the false-low rate may be large enough to affect ELO's migration recommendation.

**FM2 Path B vs. Path C distinction:**
- **Path B** — scraper exists but does not write `ecnl_verified`. Simple field addition. This spec applies directly.
- **Path C** — scraper architecture (e.g., shared generic writer, ORM mapping, auto-generated schema) prevents simple field addition. Section 2C below applies; SENTINEL escalation is required.

---

## Section 2: Proposed Code Change

### Path B: Simple Field Addition

**Target file:** `[FM2-1 grep result — file:line — fill when Presten returns results]`

If FM2-1 returned "not found," the target is the TGS scraper game INSERT function. Search for the game write path:

```bash
grep -rn "INSERT INTO games\|games\.create\|games\.upsert\|writeGame\|insertGame" . \
  --include="*.js" --include="*.ts" | grep -v test | head -20
```

The result identifies the function where the `ecnl_verified` field must be added.

**Current behavior:** The INSERT/upsert payload for TGS games does not include an `ecnl_verified` key. The column exists in the database (after April 28 ALTER TABLE) but receives the default value `FALSE` for all new rows.

**Required behavior:** TGS scraper game writes must set `ecnl_verified = TRUE` when the game belongs to an ECNL event.

**Change (≤ 5 lines — add to INSERT payload):**

```javascript
// In TGS scraper game write logic — add to INSERT/upsert payload
// Confirm variable names match actual scraper field names before applying
ecnl_verified: (
  ['tgs', 'tgs_athleteone'].includes(source) &&
  ['ecnl', 'ecnl_rl'].includes(event_tier)
),
```

**Alternatives if variable names differ:**
- If source field is named differently (e.g., `sourceId`, `scraper_type`): confirm from the FM2-1 file context and substitute.
- If `event_tier` is not yet assigned at the point of the game write: use the ECNL-identifying field available at that point (e.g., `league_id`, `org_type`). The goal is: any game where ECNL origin is determinable at write time should receive `ecnl_verified = TRUE`.

**Change type:** Additive — new key in existing INSERT/upsert payload. No schema migration required (column already exists after April 28). No modification to existing conditional logic.

**Does this modify existing behavior?** No. It adds a field that previously was not included. Existing behavior (game INSERT proceeds, game data is written) is unchanged.

---

### Path C: Structural Blocker — SQL Catch-Up Alternative

If Presten's FM2 result is Path C (architecture prevents simple field addition):

**This spec does not apply.** FM1-FM2 Contingency Action Plan Section 2 Path C covers this case.

SENTINEL must be escalated using the FM2 Path C escalation template (see `Infrastructure/FM1-FM2 Contingency Action Plan.md` Section 4). The SQL alternative:

```sql
-- Nightly catch-up UPDATE — append to run-daily.sh as post-crawl step
UPDATE games
SET ecnl_verified = TRUE
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier IN ('ecnl', 'ecnl_rl')
  AND ecnl_verified = FALSE
  AND created_at > NOW() - INTERVAL '48 hours';
```

Limitation: any window between pipeline run and catch-up UPDATE will show `ecnl_verified = FALSE` for new ECNL games. ELO's CP2 queries must account for this if they run before the catch-up UPDATE completes.

---

## Section 3: Test Plan

After applying the Path B code change:

**Smoke test 1 — Confirm field exists in code:**
```bash
grep -n "ecnl_verified" [target-file-path]
```
Expected: assignment line is present and positioned within the INSERT/upsert payload.

**Smoke test 2 — Confirm logic targets correct source/event_tier combination:**
Manually trace the logic:
- Set `source = 'tgs_athleteone'` and `event_tier = 'ecnl'` → expect `ecnl_verified = true`
- Set `source = 'gotsport_api'` and `event_tier = 'ecnl'` → expect `ecnl_verified = false` (not a TGS record)
- Set `source = 'tgs'` and `event_tier = 'ga'` → expect `ecnl_verified = false` (not an ECNL record)

If the scraper can run a dry-run against a test ECNL event URL, do so and confirm `ecnl_verified` is present in the output payload with `true` for ECNL records.

**Expected vs. actual:** After the first May 1 pipeline run, confirm:
```sql
SELECT COUNT(*) FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier IN ('ecnl', 'ecnl_rl')
  AND created_at > NOW() - INTERVAL '24 hours'
  AND ecnl_verified = TRUE;
```
Expected: count > 0 (new ECNL games have `ecnl_verified = TRUE`). If count = 0 despite new ECNL games ingested, the code change was not deployed or did not cover the right write path.

**Rollback trigger:** If post-deploy, `ecnl_verified` is set to `TRUE` on non-ECNL records (false positives), or causes an INSERT error (e.g., column type mismatch), rollback immediately.

---

## Section 4: Rollback Plan

1. **Rollback command:** `git revert [commit-hash]` — revert the specific commit that added the `ecnl_verified` field to the INSERT payload. If using a feature branch, merge the revert PR before the next scheduled pipeline run.

2. **How long:** Git revert is < 2 minutes. Redeploy (if manual) adds 5–10 minutes. Total: ≤ 15 minutes.

3. **Does rollback require another recompute?** No. Removing `ecnl_verified` from the INSERT payload does not affect ranking computation. The column will continue to exist in the schema. Any rows that received `ecnl_verified = TRUE` during the bad deployment retain their value — they are not rolled back automatically. If incorrect `TRUE` values were inserted on non-ECNL records, a correction UPDATE is needed after rollback:
   ```sql
   UPDATE games
   SET ecnl_verified = FALSE
   WHERE source NOT IN ('tgs', 'tgs_athleteone')
     AND ecnl_verified = TRUE
     AND created_at > '[deploy timestamp]';
   ```

4. **DB state recoverability:** Partial. The schema change (column addition) persists. The rollback returns the INSERT payload to its pre-change state. If incorrect `TRUE` values were written, the correction UPDATE above cleans them up. The backfill SQL Package (covering pre-May-1 records) is not affected by rollback.

5. **When to contact SENTINEL vs. self-remediate:**
   - Self-remediate: rollback if smoke test fails, or if post-deploy count check shows 0 new ECNL records with `ecnl_verified = TRUE`
   - Contact SENTINEL: if incorrect `ecnl_verified = TRUE` was written to non-ECNL records at scale (> 100 records), or if the INSERT path error corrupted game records (e.g., null game IDs, missing required fields)

---

## Section 5: SENTINEL Authorization Request

Pre-written for use if FM2 Path B is confirmed and FORGE is requesting authorization to deploy:

> **To:** SENTINEL
> **From:** FORGE
> **Re:** FM2 Path B Confirmed — Code Change Authorization Request
>
> Presten filed FM2 Path B: the TGS scraper exists and writes game data but does not include `ecnl_verified` in the INSERT payload. The April 28 backfill covers existing records. Without this code change, every new ECNL game after May 1 will have `ecnl_verified = FALSE`, degrading ELO's CP2 evidence quality over time.
>
> **File:** `[FM2-1 file:line from Pipeline Code Review Results.md]`
> **Change type:** Additive — new field in INSERT payload. Does not modify existing behavior.
> **Change size:** ≤ 5 lines.
> **Test plan:** Grep confirmation + first-run SQL count check on May 1 after pipeline executes.
> **Rollback:** `git revert [commit-hash]`, ≤ 15 minutes. DB correction UPDATE available if false-positives found.
> **Deployment window:** Before May 1 launch (preferred). If not possible before May 1, deploy on May 2 and run supplemental backfill for May 1 ECNL games.
>
> **Requesting SENTINEL authorization to deploy this change before May 1.**

---

## References

- `Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md` — FM2-1/FM2-2/FM2-3 result fields
- `Infrastructure/FM1-FM2 Contingency Action Plan.md` — FM2 branch table and Path C escalation template
- `Infrastructure/Pipeline Code Review Results.md` — FM2 path confirmation source
- `Infrastructure/ECNL ecnl_verified — Backfill SQL Package.md` — covers existing records; this spec covers forward ingestion
- `Infrastructure/ecnl_verified — Pipeline Write-Time Verification Spec.md` — post-deploy verification spec

*FORGE — 2026-04-27*
