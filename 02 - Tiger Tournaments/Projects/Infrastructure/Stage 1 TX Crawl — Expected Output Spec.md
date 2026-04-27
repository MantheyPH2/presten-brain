---
title: Stage 1 TX Crawl — Expected Output Spec
tags: [infrastructure, gotsport, stage1, texas, pipeline, expected-output, evo-draw]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: pre-crawl estimates — update with actuals when Stage 1 runs
---

# Stage 1 TX Crawl — Expected Output Spec

> **Purpose:** Pre-crawl estimates and pass/fail criteria for the Stage 1 Texas GotSport pilot crawl. Written before Stage 1 executes so that when results arrive, SENTINEL and Presten have an immediate baseline to assess against.

---

## Section 1: TX Org-ID Scope

Stage 1 is a single-entity pilot crawl against the Texas state association. The org-ID requires browser confirmation before Stage 1 can execute.

| Org-ID | Entity Name | Type | Game Volume Estimate (per season) | Source of Estimate |
|--------|-------------|------|----------------------------------|-------------------|
| pending-browser-lookup | Texas Youth Soccer Association (TYSA) | USYS State Association | 30,000–60,000 games/year | Texas: ~75,000 registered teams (USYS 2023 data); GotSport coverage ≈50–80% of teams; ~8 games/team/season |

**Scope note:** Stage 1 is intentionally narrow — one org-ID, one state association — to validate crawl configuration and pipeline behavior before broader expansion. The TYSA org-ID covers all USYS-affiliated Texas leagues including Texas State League, Club Texas, TYSA State Cup, and affiliated tournaments.

**Blocker:** TYSA's GotSport org-ID is `pending-browser-lookup`. Stage 1 cannot execute until Presten confirms the org-ID via the browser session at `system.gotsport.com`. See `Infrastructure/GotSport Browser Lookup Session Prep Package.md`.

---

## Section 2: Aggregate Expected Range

### Methodology

Texas youth soccer volume is estimated from:
- USYS Texas registered team count: ~75,000 teams (2023 USYS annual report, conservative read)
- GotSport platform penetration for Texas: estimated 50–80% of USYS-registered games appear on GotSport (vs. club-managed systems)
- Games per team per season: ~8 games (competitive season average, including fall + spring)
- Season fraction for April: ~60% of annual games played by end of April (spring season underway, fall season complete, summer tournaments not yet started)

### Volume Estimates

| Estimate Tier | Annual Games | April-Fraction Games (60%) |
|---------------|-------------|---------------------------|
| Low (50% GotSport penetration) | 30,000 | 18,000 |
| Expected (65% penetration) | 39,000 | 23,400 |
| High (80% penetration) | 48,000 | 28,800 |

**Formula:**  
`Expected TX games accessible via crawl = (75,000 teams × avg_games_per_team × GotSport_penetration) × season_fraction`

**Key uncertainties:**
- Org-ID scope: a single TYSA org-ID may capture all affiliated teams or only state association-hosted events. If TYSA on GotSport hosts state cups but not club league games, the actual count will be substantially lower (5,000–15,000 range).
- DB overlap: some TX games may already be in the database from existing scrapers (TGS source). The new game count will be new-games-only after dedup; gross crawl volume will be higher.

**Working expected count (point estimate):** ~15,000–25,000 new games accessible (net of existing DB records).

---

## Section 3: Pass/Fail Criteria for Stage 1

| Result Category | Criterion | Interpretation |
|----------------|-----------|---------------|
| GREEN — healthy | New games ingested ≥ 5,000 | Pipeline reached GotSport, parsed data, handled dedup. Stage 2 authorized (pending other gates). |
| YELLOW — investigate | New games between 500 and 4,999 | Partial success — crawl may be scoped to state-cup events only, or GotSport org-ID may be narrower than expected. Identify which sub-sections of TYSA data are accessible before authorizing Stage 2. |
| RED — failure | New games < 500, or 0 games from TYSA org-ID | Stage 1 failed. Do not proceed to Stage 2. Diagnose: confirm org-ID, check crawl logs for 4xx/5xx errors, verify config syntax. |

**GREEN floor rationale:** 5,000 is ~10% of the low-end annual estimate and ~25% of the April-season low estimate. A healthy crawl hitting even state cups only should exceed this threshold.

**Secondary criterion — ecnl_verified population rate:**  
After the crawl, run S4-4 (from `Infrastructure/April 29 — Section 4 Source Baseline Query Package.md`). If ecnl_verified population rate is 0% for ECNL-tagged games post-crawl, flag to SENTINEL. This indicates the ecnl_verified backfill did not run or new ECNL games are not being tagged at write time.

**Secondary criterion — deduplication:**  
```sql
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1;
```
Expected: 0 rows. Any duplicate rows are a dedup pipeline failure — flag to SENTINEL before running Stage 2.

---

## Section 4: Post-Crawl Verification Queries

Run these immediately after the Stage 1 crawl completes.

**Q1 — TX Game Count by Org-ID**
```sql
-- Game count by source org_id, filtered to last 24 hours of ingestion
SELECT source_org_id, COUNT(*) AS game_count
FROM games
WHERE source = 'gotsport'
  AND created_at > NOW() - INTERVAL '24 hours'
GROUP BY source_org_id
ORDER BY game_count DESC;
```
Expected: One row for TYSA's org-ID showing game count. All active TX org-IDs should show > 0.

**Q2 — Crawl Recency Check**
```sql
-- Most recent game_date ingested per org_id
SELECT source_org_id, MAX(game_date) AS latest_game_date, MIN(game_date) AS earliest_game_date
FROM games
WHERE source = 'gotsport'
  AND created_at > NOW() - INTERVAL '24 hours'
GROUP BY source_org_id;
```
Expected: `latest_game_date` in 2025–2026 range. `earliest_game_date` may go back several years if full history was captured.

**Q3 — Source Error Check**
```sql
-- Any source-level errors or null org_id entries from this crawl run
SELECT source_org_id, COUNT(*) AS count
FROM games
WHERE source = 'gotsport'
  AND created_at > NOW() - INTERVAL '24 hours'
  AND (source_org_id IS NULL OR source_org_id = '')
GROUP BY source_org_id;
```
Expected: 0 rows with null org_id.

---

## Section 5: May 1 Expected Count Inheritance

When Stage 1 runs and produces a GREEN result:
1. Record the actual new game count from Q1.
2. Open `Infrastructure/May 1 Deployment — Day-Of Runbook.md`.
3. Replace the placeholder `"Expected game count: pending Stage 1 results"` with:  
   `"Expected new games per run: [Stage 1 actual count] ± 20% tolerance. Source: Stage 1 results [date]."`
4. This becomes the GREEN threshold for the May 1 first-run validation checklist.

**If Stage 1 does NOT run before May 1:**  
Use the low estimate (5,000 games) as the GREEN minimum threshold. Flag in the May 1 briefing: "Expected game count is pre-crawl estimate — Stage 1 did not execute before May 1. Adjust GREEN threshold after first actual run confirms volume."

---

## Section 6: Stage 1 Exit Conditions

Stage 1 is complete when **all three conditions are met:**

```
[ ] GREEN game count confirmed (≥ 5,000 new games ingested)
[ ] Dedup check passed (0 duplicate rows in Q3-style check)
[ ] Second crawl run confirms no new games on re-run (count delta = 0 or near 0)
```

Second crawl requirement: Run the crawl again after the first run completes. The delta between run 1 and run 2 should be near zero (only games from the last few hours, not a re-ingestion of all games). If delta is large on run 2, the pipeline is not deduplicating correctly — **do not proceed to Stage 2 until resolved.**

---

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — TYSA org-ID status (Section 1)
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — org-ID confirmation path
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — inherits expected count from this spec (Section 5)
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — GREEN criteria that depend on expected count
- `Infrastructure/April 29 — Section 4 Source Baseline Query Package.md` — ecnl_verified check (secondary criterion)
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — TX crawl validation context
- `task-2026-04-25-stage1-backfill-results-report.md` — results report to file after Stage 1 executes

*FORGE — 2026-04-27*
