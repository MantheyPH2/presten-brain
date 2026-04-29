---
title: Data Pipeline
aliases: [Pipeline, Processing Chain, run-pipeline.sh]
tags: [pipeline, processing, normalization, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Data Pipeline

The Evo Draw data pipeline transforms raw scraped data into ranked, deduplicated, and normalized game records. It consists of **9 sequential steps** that must run in order.

---

> [!warning] Run Order Matters
> These steps have dependencies. Running them out of order will produce incorrect results. Use `run-pipeline.sh` for the correct sequence, or `run-overnight.sh` for a full batch including scrapers.

---

## Pipeline Steps

### Step 1: cleanup-and-normalize.js
- Purges bad/invalid events
- Normalizes `round` values (see [[Parsing Rules]] for detection order)
- Normalizes `game_status` (completed, scheduled, cancelled, etc.)
- Normalizes `season` assignment (Aug-Dec → "YYYY-YYYY+1", Jan-Jul → "YYYY-1-YYYY")
- Removes obvious junk data

### Step 2: backfill-age-gender.js
- Parses `age_group` and `gender` from division text
- Uses the [[Parsing Rules|parsing priority rules]]
- Handles standard formats: U10, U-10, 10U, Sub 17, Under 14

### Step 3: final-backfill.js
- Edge cases not caught by Step 2
- Multi-age divisions: U13/U14 → takes lower (U13)
- Compact formats: 13UB → U13 Boys, 11UG → U11 Girls
- Spanish formats: Sub 17 → U17

### Step 4: backfill-all-games.js
- Universal backfill pass across ALL sources
- Catches anything missed by source-specific backfills
- Applies [[Parsing Rules|sibling inference]] (95%+ same gender → apply to remaining)

### Step 5: fix-api-gender.js
- Targeted fix for null-gender games from the [[GotSport API Scraper]]
- API data sometimes lacks explicit gender markers
- Uses team name patterns and event context to infer

### Step 5.5: archive-inactive-teams.js *(awaiting first production run — see [[Archive Workflow]])*
- Identifies teams where last game >180 days ago AND event ended >90 days ago AND no sync activity in 30 days
- Safety guard: excludes teams with any game in the last 90 days across any event
- Sets `archived_at = NOW()` on confirmed candidates; outputs a candidate log before any UPDATE runs
- Schema migration (`archived_at TIMESTAMP NULL`) **executed on production 2026-04-22** — column exists and indexed
- GROUP BY bug (duplicate team rows) fixed in spec 2026-04-27 — fix must be applied to script before first dry-run
- Script runs via `ARCHIVE_STEP=true bash run-pipeline.sh` (off by default; run weekly during season)
- Runs BEFORE dedup so archived teams are excluded from dedup pass and team merges loader (Phase 1 ranking engine)

### Step 6: dedup-and-link-teams.js
- **Cross-source dedup:** Same game in multiple sources → keep highest priority per [[Dedup Strategy]]
- **Within-source dedup:** Exact duplicate records → keep best version
- Sets `is_hidden=true` on duplicate records
- Sets `duplicate_of` to point to the canonical version
- See [[Dedup Strategy]] for full logic

### Step 7: match-teams-cross-platform.js
- Links teams across platforms using:
  - Coach names (strongest signal)
  - Fuzzy name matching
  - Geographic proximity
- Populates the `team_merges` table (27,820 entries)
- See [[Team Name Matching]] for GotSport-specific challenges

### Step 8: compute-rankings.js
- The [[Ranking Engine]] — Elo/Glicko v7
- 9+1 phases from team merges through H2H enforcement
- Outputs to the `rankings` table
- See [[Ranking Engine]] for complete phase documentation

### Step 9: audit-data.js
- Data quality report
- Checks completeness of all key fields
- Reports on [[Data Quality|known gaps]]: null gender, null age_group, etc.
- Generates summary statistics

---

## Shell Scripts

### run-pipeline.sh
Runs Steps 1-9 in sequence. Use for daily processing after scraping completes.

```bash
bash run-pipeline.sh
```

### run-overnight.sh
Full batch: runs all scrapers first, then the pipeline. Designed for unattended overnight execution.

```bash
bash run-overnight.sh
```

---

## Data Flow

```
Raw Data (scrapers)
  ↓
cleanup-and-normalize.js     → clean events, normalize fields
  ↓
backfill-age-gender.js       → parse age/gender from text
  ↓
final-backfill.js            → edge case parsing
  ↓
backfill-all-games.js        → universal backfill
  ↓
fix-api-gender.js            → targeted API gender fix
  ↓
dedup-and-link-teams.js      → deduplicate games
  ↓
match-teams-cross-platform.js → link teams across sources
  ↓
compute-rankings.js          → compute Elo rankings
  ↓
audit-data.js                → quality report
```

---

## Related Notes
- [[Data Sources Overview]] — where the raw data comes from
- [[Dedup Strategy]] — Step 6 detailed logic
- [[Parsing Rules]] — Steps 2-5 detailed rules
- [[Ranking Engine]] — Step 8 detailed algorithm
- [[Data Quality]] — Step 9 output metrics
- [[Database Schema]] — tables written by the pipeline
- [[Daily Note - Evo Draw]] — template for tracking pipeline runs
