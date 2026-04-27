---
title: "Stage 1 TX Crawl — Post-Run Validation Checklist"
tags: [infrastructure, gotsport, stage1, texas, pipeline, validation, forge, evo-draw]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: ready — execute immediately after Stage 1 TX crawl completes
---

# Stage 1 TX Crawl — Post-Run Validation Checklist

> **When to use:** Open this document the moment Stage 1 TX crawl completes. Work through it linearly. File the verdict in the next FORGE briefing.
>
> **Expected crawl duration:** 4–12 hours depending on TYSA org-ID scope. Do not start validation until the crawl process has exited cleanly.
>
> **GREEN/YELLOW/RED thresholds:** Defined in [[Stage 1 TX Crawl — Expected Output Spec]] Section 3.

---

## Pre-Validation Gate

Before running any queries:

```
[ ] Stage 1 TX crawl confirmed complete (pipeline log shows clean exit, OR Presten confirmed completion)
[ ] Expected crawl time elapsed (minimum 2 hours after launch before assuming it's done)
[ ] TYSA org-ID confirmed from browser session and recorded in Org-ID Master Reference
[ ] psql session open
```

**If the crawl did not exit cleanly (error exit, timeout, crash):** Do NOT proceed to validation. File an immediate escalation to Presten with the error log. The validation queries are meaningless against a partial crawl.

---

## Section 2: Validation Queries

Run in order. Record exact output. Replace `[STAGE_1_TX_ORG_IDS]` with the confirmed TYSA org-ID (and any additional Stage 1 TX org-IDs) from `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`.

---

**V1: New game count from Stage 1 org-IDs**

```sql
-- Count games ingested from TYSA and any other Stage 1 TX org-IDs
-- Replace [STAGE_1_TX_ORG_IDS] with confirmed values, e.g. IN (12345) or IN (12345, 67890)
SELECT source_org_id, COUNT(*) AS game_count
FROM games
WHERE source = 'gotsport'
  AND source_org_id IN ([STAGE_1_TX_ORG_IDS])
ORDER BY game_count DESC;
```

*Record:* Per-org-ID count and total. This is the primary Stage 1 pass/fail metric.

Also capture the rolling 24-hour count:
```sql
SELECT source_org_id, COUNT(*) AS game_count
FROM games
WHERE source = 'gotsport'
  AND source_org_id IN ([STAGE_1_TX_ORG_IDS])
  AND created_at > NOW() - INTERVAL '24 hours'
ORDER BY game_count DESC;
```

*Total new games (record here):* ________________

---

**V2: Team match rate check**

```sql
-- What percentage of Stage 1 games have a matched team_id (not null)?
SELECT
  COUNT(*) AS total_stage1_games,
  COUNT(team_id) AS matched_games,
  ROUND(100.0 * COUNT(team_id) / COUNT(*), 1) AS match_pct
FROM games
WHERE source = 'gotsport'
  AND source_org_id IN ([STAGE_1_TX_ORG_IDS]);
```

*Record:* Match percentage ______%.

**FLAG if match_pct < 80%** — low match rate indicates TX team name format differs from existing team records. Dedup and name-matching may need tuning before Stage 2 proceeds with TX data.

---

**V3: Duplicate game check**

```sql
-- Check for duplicate games: same teams, same date, same event, multiple rows
SELECT game_date, home_team_id, away_team_id, event_id, COUNT(*) AS instances
FROM games
WHERE source = 'gotsport'
  AND source_org_id IN ([STAGE_1_TX_ORG_IDS])
GROUP BY game_date, home_team_id, away_team_id, event_id
HAVING COUNT(*) > 1
LIMIT 20;
```

*Record:* Number of duplicate rows returned ______.

**Expected: 0 rows.** Any duplicates indicate a dedup pipeline failure. **Do not authorize Stage 2 if duplicates are present** — investigate and resolve before expanding the crawl scope.

---

**V4: Date range sanity**

```sql
-- What date range do Stage 1 TX games cover?
SELECT
  MIN(game_date) AS earliest_game,
  MAX(game_date) AS latest_game,
  COUNT(*) AS total_games
FROM games
WHERE source = 'gotsport'
  AND source_org_id IN ([STAGE_1_TX_ORG_IDS]);
```

*Record:* Earliest ____________, Latest ____________.

**Expected:** Games span at least the 2025–2026 season. `latest_game_date` should be within 30–60 days of today (spring season underway).

**FLAG if all games are older than 2 years** — the org-ID may have returned historical-only data or the date filter is set too conservatively.

---

## Section 3: Threshold Assessment

Fill in after running V1–V4:

| Check | Expected | Actual | Status |
|-------|----------|--------|--------|
| Total new games (V1) | ≥ 5,000 (GREEN) | | RED / YELLOW / GREEN |
| Team match rate (V2) | ≥ 80% | | OK / FLAG |
| Duplicate rows (V3) | 0 | | OK / FLAG |
| Date range includes 2025–26 season (V4) | Yes | | OK / FLAG |

**Overall Stage 1 TX Verdict:**

- **GREEN** (total games ≥ 5,000 AND V2–V4 all OK): Stage 1 confirmed healthy. Proceed to Stage 2 planning. Update May 1 Day-Of Runbook with Stage 1 actual count.
- **YELLOW** (total games 500–4,999, no other flags): Crawl reached GotSport but scope may be narrower than expected (state cups only, not club leagues). Flag to SENTINEL. Presten reviews TYSA org-ID scope before authorizing Stage 2.
- **RED** (total games < 500, OR any FLAG from V2 or V3): Hold Stage 2. Escalate to Presten with the specific check that failed and the raw query output.

**FORGE verdict (fill in):** ________________

---

## Section 4: Post-Validation Actions

Complete regardless of verdict:

1. **Update Expected Output Spec:**
   - Open `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md`
   - Fill in Section 4 (Post-Crawl Verification Queries) with actual query outputs from V1–V4 above
   - Record actual game count alongside the pre-crawl estimate

2. **Update Org-ID Master Reference:**
   - Open `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`
   - Move TYSA from Section 2 (Pending) to Section 1 (Active)
   - Record: Stage 1 game count, validation date, GREEN/YELLOW/RED status

3. **Update May 1 Day-Of Runbook (if GREEN):**
   - Open `Infrastructure/May 1 Deployment — Day-Of Runbook.md`
   - Replace placeholder "Expected game count: pending Stage 1 results" with:
     `"Expected new games per run: [Stage 1 actual count] ± 20% tolerance. Source: Stage 1 results [date]."`

4. **File FORGE briefing** with:
   - Stage 1 TX result: GREEN / YELLOW / RED
   - Total new games: [N]
   - Team match rate: [N]%
   - Duplicate count: [N]
   - Date range: [earliest] to [latest]
   - Any flags and escalation status

5. **Notify SENTINEL:**
   > "Stage 1 TX validation complete. Verdict: [X]. [N] new games. Match rate: [N]%. Flags: [list or 'none']. Stage 2 authorization: [proceed / hold]."

---

## Secondary Criteria (Run If GREEN)

**ecnl_verified check** (required before Stage 2 authorization):

```sql
-- Confirm ecnl_verified is populating correctly for any new ECNL-tagged games
SELECT ecnl_verified, COUNT(*) AS count
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND created_at > NOW() - INTERVAL '48 hours'
GROUP BY ecnl_verified;
```

**Expected:** All rows show `ecnl_verified = TRUE` (if backfill has run).
**FLAG if ecnl_verified = FALSE for new games:** Write-time population is not working. See `Infrastructure/ecnl_verified — Pipeline Write-Time Verification Spec.md`.

**Second crawl re-run check** (Stage 1 exit condition):

After confirming GREEN, run the crawl a second time and verify the delta is near-zero:

```sql
-- Delta between run 1 and run 2 — should be near 0 (only very recent games)
SELECT COUNT(*) AS new_games_run2
FROM games
WHERE source = 'gotsport'
  AND source_org_id IN ([STAGE_1_TX_ORG_IDS])
  AND created_at > '[Run 2 start timestamp]';
```

**Expected:** < 50 new games on run 2 (only games from the few hours between runs, not a full re-ingest).
**FLAG if delta is large (> 200 games):** Dedup is not working correctly — do not proceed to Stage 2.

---

## Stage 1 Exit Conditions

Stage 1 is complete and Stage 2 may proceed when **all three conditions are met:**

```
[ ] GREEN game count confirmed (≥ 5,000 new games ingested from V1)
[ ] Dedup check passed: V3 = 0 duplicate rows AND second-run delta is near-zero
[ ] ecnl_verified secondary criterion passed (if applicable)
```

File the exit condition confirmation in the FORGE briefing before notifying SENTINEL to open Stage 2.

---

## References

- [[Stage 1 TX Crawl — Expected Output Spec]] — GREEN/YELLOW/RED thresholds and pre-crawl estimates
- [[GotSport Org-ID Master Reference — April 2026]] — TYSA org-ID confirmation and status update
- [[Browser Session — Post-Receipt Org-ID Action Plan]] — org-ID confirmation that unblocked Stage 1
- [[May 1 Deployment — Day-Of Runbook]] — inherits Stage 1 actual game count for expected run volume
- [[ecnl_verified — Pipeline Write-Time Verification Spec]] — secondary criterion for ecnl_verified accuracy

*FORGE — 2026-04-27*
