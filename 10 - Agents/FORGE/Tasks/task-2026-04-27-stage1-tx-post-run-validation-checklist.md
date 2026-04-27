---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 TX Crawl — Post-Run Validation Checklist.md"
---

# Task: Stage 1 TX Crawl — Post-Run Validation Checklist

## Context

FORGE filed `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` this session, defining GREEN/YELLOW/RED thresholds for Stage 1 TX crawl output (GREEN: ≥ 5,000 new games; YELLOW: 1,000–4,999; RED: < 1,000). The spec says what numbers to expect. What it does not contain is the structured sequence FORGE executes when the crawl completes — queries to run, checks to make, decisions to record.

Stage 1 TX cannot run until the browser session confirms the TYSA org-ID. The browser session may happen any day from April 27 onward. When it does, FORGE's post-receipt action plan kicks in — and if TYSA is one of the confirmed org-IDs, Stage 1 starts within hours. FORGE should not enter that validation window without a pre-written checklist.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 TX Crawl — Post-Run Validation Checklist.md`

---

### Section 1: Pre-Validation Gate

Before FORGE starts validation queries, confirm:

- [ ] Stage 1 TX crawl has completed (FORGE observed pipeline log exit or Presten confirmed completion)
- [ ] Sufficient time has elapsed for ingestion to complete (FORGE states the expected crawl + ingest time based on the Expected Output Spec)
- [ ] psql session is open

If the crawl did not complete successfully (error exit, timeout), FORGE does not proceed to validation — escalate to Presten with the error log immediately.

---

### Section 2: Validation Queries (Copy-Paste Ready)

Run in order. Record output exactly.

**V1: New game count from Stage 1 org-IDs**
```sql
-- Count games ingested from TYSA and any other Stage 1 TX org-IDs
-- Replace org_id values with confirmed Stage 1 TX org-IDs from Master Reference
SELECT org_id, COUNT(*) AS game_count
FROM games
WHERE source = 'gotsport'
  AND org_id IN ([STAGE_1_TX_ORG_IDS])
ORDER BY game_count DESC;
```
*Record:* per-org-ID count and total. Compare against Expected Output Spec thresholds.

**V2: Team match rate check**
```sql
-- What percentage of Stage 1 games have a matched team_id (not null)?
SELECT
  COUNT(*) AS total_stage1_games,
  COUNT(team_id) AS matched_games,
  ROUND(100.0 * COUNT(team_id) / COUNT(*), 1) AS match_pct
FROM games
WHERE source = 'gotsport'
  AND org_id IN ([STAGE_1_TX_ORG_IDS]);
```
*Record:* match percentage. Flag if < 80% — low match rate may indicate team name format differs from expected and requires name-matching tuning.

**V3: Duplicate game check**
```sql
-- Check for duplicates: same teams, same date, same event, multiple rows
SELECT game_date, home_team_id, away_team_id, event_id, COUNT(*) AS instances
FROM games
WHERE source = 'gotsport'
  AND org_id IN ([STAGE_1_TX_ORG_IDS])
GROUP BY game_date, home_team_id, away_team_id, event_id
HAVING COUNT(*) > 1
LIMIT 20;
```
*Record:* number of duplicate rows returned. Expected: 0. If > 0, flag to Presten for dedup investigation before Stage 2 proceeds.

**V4: Date range sanity**
```sql
-- What date range do Stage 1 TX games cover?
SELECT
  MIN(game_date) AS earliest_game,
  MAX(game_date) AS latest_game,
  COUNT(*) AS total_games
FROM games
WHERE source = 'gotsport'
  AND org_id IN ([STAGE_1_TX_ORG_IDS]);
```
*Record:* earliest and latest game dates. Expected: games span at least the current season (fall 2025 – spring 2026). If all games are older than 2 years, the org-ID may have returned historical-only data — flag for review.

---

### Section 3: Threshold Assessment

Fill in after running V1–V4:

| Check | Expected | Actual | Status |
|-------|----------|--------|--------|
| Total new games (V1) | ≥ 5,000 (GREEN) | | RED / YELLOW / GREEN |
| Team match rate (V2) | ≥ 80% | | OK / FLAG |
| Duplicate rows (V3) | 0 | | OK / FLAG |
| Date range includes 2025–26 season (V4) | Yes | | OK / FLAG |

**Overall Stage 1 TX Verdict:**
- GREEN (all checks OK + ≥ 5,000 games): Proceed to Stage 2 planning.
- YELLOW (game count 1,000–4,999, no other flags): Flag to SENTINEL. Presten reviews org-ID scope before Stage 2.
- RED (< 1,000 games, or any FLAG from V2–V4): Hold Stage 2. Escalate to Presten with specific check that failed.

---

### Section 4: Post-Validation Actions

Regardless of verdict:

1. Update `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` Section 4 with actual counts
2. Update `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`: Stage 1 TX org-IDs → Active, with game count and validation date
3. File FORGE briefing with: Stage 1 TX result (GREEN/YELLOW/RED), new game count, team match rate, any flags
4. Notify SENTINEL: "Stage 1 TX validation complete. Verdict: [X]. [N] games added. Flags: [list or 'none']."

---

## Why This Matters

Stage 1 TX is the first expansion of the GotSport crawl scope. Its success determines whether Stage 2 proceeds on schedule (target: post-May-1 pipeline stability confirmed). A completed crawl without structured validation is not a confirmed success — it is an unexamined result. FORGE needs this checklist ready before the browser session runs so that validation starts immediately when Stage 1 completes, not after FORGE designs the process under time pressure.

## Definition of Done

- File at deliverable path
- All 4 validation queries are copy-paste ready (STAGE_1_TX_ORG_IDS placeholder is the only fill-in, added when org-IDs are confirmed)
- Threshold assessment table is pre-populated with expected values
- Post-validation actions are actionable without additional design
- FORGE can open this file immediately after Stage 1 TX crawl completes and follow it top to bottom

## References

- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — defines GREEN/YELLOW/RED thresholds this checklist operationalizes
- `Infrastructure/Browser Session — Post-Receipt Org-ID Action Plan.md` — org-ID confirmation that unblocks Stage 1
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — master reference updated post-validation
