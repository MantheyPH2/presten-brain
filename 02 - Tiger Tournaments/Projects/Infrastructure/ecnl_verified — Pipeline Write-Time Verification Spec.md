---
title: "ecnl_verified — Pipeline Write-Time Verification Spec"
tags: [infrastructure, sql, ecnl, ecnl-verified, pipeline, write-time, forge, evo-draw]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: ready — verify Section 1 finding after May 1 first run
---

# ecnl_verified — Pipeline Write-Time Verification Spec

> **Purpose:** Confirm that new ECNL games ingested after May 1 receive `ecnl_verified = TRUE` at write time. The [[ECNL ecnl_verified — Backfill SQL Package]] covers historical games; this spec covers ongoing population.
>
> **Execution window:** Verification query runs 3–5 days after May 1 (i.e., May 4–6). Code change (if needed) deploys before May 16 DSS.

---

## Section 1: Write Path Identification

**ECNL ingestion paths confirmed from Backfill SQL Package:**

- `source = 'tgs'` — TGS scraper
- `source = 'tgs_athleteone'` — AthleteOne variant of TGS scraper
- Both combined with `event_tier = 'ecnl'` to identify ECNL-classified games

**Write-time population status:**

**FINDING: Write path exists — ecnl_verified population status unconfirmed**

As of April 27, the `ecnl_verified` column was added on April 28 via `ALTER TABLE games ADD COLUMN ecnl_verified BOOLEAN DEFAULT FALSE`. The ALTER TABLE sets the default for new rows. Whether the TGS scraper (the active ECNL ingestion path) explicitly sets `ecnl_verified = TRUE` at insert time is **not yet confirmed** — the April 28 pipeline code update may or may not have included this.

**Investigation steps:**
1. After May 1 first pipeline run, run the Section 3 verification query.
2. If new May 1 ECNL games show `ecnl_verified = TRUE` → scraper already sets the value. No code change needed.
3. If new May 1 ECNL games show `ecnl_verified = FALSE` (the `DEFAULT FALSE` value) → scraper is not setting `ecnl_verified` at insert. Section 2 code change is required.

**File path of TGS scraper write function (for Presten to check directly):**
```
[TBD: confirm file path from codebase — expected location: scrapers/tgs/ or similar]
The INSERT/UPSERT function should be found in the TGS scraper's DB write module.
Look for: ecnl_verified in the column list of INSERT INTO games (...) VALUES (...)
```

---

## Section 2: Required Code Change (If Section 1 Confirms ecnl_verified Not Set)

If the May 1 verification query shows new ECNL games have `ecnl_verified = FALSE`, the following change is required in the TGS scraper:

**Change type:** Application-layer only. No schema migration required — column already exists.

**What to change:**

In the TGS scraper's database write function (INSERT or UPSERT for `games`), add `ecnl_verified` to the column list with conditional logic:

```javascript
// Pseudocode — adapt to actual scraper language/ORM
// In the game INSERT/UPSERT block:
const ecnl_verified = (event_tier === 'ecnl') ? true : false;

// Add to INSERT column list:
// ecnl_verified: ecnl_verified
```

Or in SQL form:
```sql
INSERT INTO games (
  ...,
  ecnl_verified
) VALUES (
  ...,
  CASE WHEN event_tier = 'ecnl' THEN TRUE ELSE FALSE END
)
ON CONFLICT (game_id) DO UPDATE SET
  ...,
  ecnl_verified = EXCLUDED.ecnl_verified;
```

**This change is additive-only:** The column already exists with a default value. Adding explicit population at write time does not change existing game records (the backfill covers those). It only ensures new games are tagged correctly. Safe to deploy without a separate schema migration.

**If code change is confirmed unnecessary:** State in briefing: "ecnl_verified write-time spec complete. No code change needed — scraper sets value at write time. Confirmed May [date]."

---

## Section 3: Verification Query

Run 3–5 days after May 1 (i.e., May 4–6) to confirm that new ECNL games have ecnl_verified populated correctly.

```sql
-- ecnl_verified write-time verification
-- Run on May 4–6 after first daily pipeline runs have completed
SELECT
  source,
  COUNT(*) AS ecnl_games_since_may1,
  SUM(CASE WHEN ecnl_verified IS TRUE THEN 1 ELSE 0 END) AS verified_count,
  SUM(CASE WHEN ecnl_verified IS FALSE OR ecnl_verified IS NULL THEN 1 ELSE 0 END) AS unverified_count,
  ROUND(100.0 * SUM(CASE WHEN ecnl_verified IS TRUE THEN 1 ELSE 0 END) / COUNT(*), 1) AS pct_verified
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND created_at >= '2026-05-01'
GROUP BY source
ORDER BY ecnl_games_since_may1 DESC;
```

**Expected result:** `pct_verified = 100.0` for all ECNL sources.

**Failure condition:** If `pct_verified < 100.0` (or = 0.0), the scraper is not setting `ecnl_verified` at write time. Execute Section 2 code change immediately — deploy before May 16 DSS.

> [!note] If no ECNL games appear since May 1
> If `ecnl_games_since_may1 = 0`, this may mean: (a) no ECNL games were played/scraped in this window, or (b) the TGS scraper was not included in the May 1 pipeline deployment. Check the pipeline cron scope before concluding there's a write-time problem.

---

## Section 4: Deployment Timeline Recommendation

| Scenario | Action | Deadline |
|---------|--------|----------|
| May 1 games show `ecnl_verified = TRUE` | No code change needed. Close this spec. | — |
| May 1 games show `ecnl_verified = FALSE` | Implement Section 2 code change. Deploy before May 16. | 2026-05-15 |
| No ECNL games found since May 1 | Investigate TGS scraper scope in cron. Not a write-time issue — different problem. | ASAP |
| Code change deployed; May 4–6 query shows 100% | Confirmed working. Note in briefing and close. | — |

**Hard deadline for code change:** May 15, 2026. Any ECNL games written after April 28 with `ecnl_verified = FALSE` will be incorrectly classified in ELO's June migration evidence baseline. The migration go/no-go is June 2026 — write-time accuracy for May–June games is required for ELO's CP2 through CP7 evidence chain.

---

## Definition of Done Checklist

```
[ ] Section 3 verification query run (on May 4–6)
[ ] Result recorded: pct_verified = ______%
[ ] If pct_verified < 100%: Section 2 code change identified and filed with Presten
[ ] If code change deployed: re-run Section 3 query to confirm fix
[ ] FORGE briefing note: "ecnl_verified write-time spec: [NO CHANGE NEEDED / CODE CHANGE DEPLOYED]. 
    pct_verified = [N]%. Deploy by: [date]."
```

---

## References

- [[ECNL ecnl_verified — Backfill SQL Package]] — the one-time backfill this spec extends to ongoing ingestion
- [[April 29 — FORGE Execution Checklist]] — Step 6 (backfill runs April 29; write-time verification is the follow-on)
- [[April 29 — Section 4 Source Baseline Query Package]] — S4-4 monitors ecnl_verified population rate
- `task-2026-04-26-ecnl-migration-evidence-collection.md` — ELO CP2 checkpoint depends on ecnl_verified accuracy
- [[Rankings/ECNL Migration Evidence Campaign]] — migration gate context (June 2026)

*FORGE — 2026-04-27*
