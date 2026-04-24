---
title: GotSport Org-ID Expansion — Test Plan & Pre-Execution Report
tags: [report, gotsport, data-sources, usys, state-cup, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: planning complete — execution requires DB + scraper access
priority: high
---

# GotSport Org-ID Expansion — Test Plan & Pre-Execution Report

**Analyst:** FORGE  
**Date:** 2026-04-24  
**Task:** `task-2026-04-24-gotsport-org-id-expansion.md`  
**Due:** 2026-04-25  
**Context:** USYS State Cup research (2026-04-23) confirmed that ~80–85% of USYS state results are on GotSport, but unranked-division teams never appear in the rankings endpoint used for team discovery. Adding explicit state association org_ids to the crawl scope is the highest-impact pre-DSS data expansion: estimated **20,000–35,000 new games**.

---

## Step 1: Target Org IDs — Research Findings

Based on the USYS State Cup Source Research and GotSport's URL patterns, the following state associations are the primary targets. GotSport org IDs appear in event URLs as `gotsport.com/games/schedule?org={id}` or in API calls as `/api/v1/team_ranking_data?org_id={id}`.

### Confirmed-GotSport States (Priority 1 — Highest Volume)

| State Association | Abbreviation | GotSport? | Est. Teams | Org ID Status | Notes |
|-------------------|-------------|-----------|------------|----------------|-------|
| Texas Soccer (STYSA) | TX | Yes | Very Large | **Needs lookup** | Largest youth soccer state by volume |
| Florida Youth Soccer | FL | Yes | Large | **Needs lookup** | GotSport-primary per USYS research |
| Eastern NY Youth Soccer (ENYYSA) | NY | Yes | Large | **Needs lookup** | Metro NY — high elite team density |
| NJ Youth Soccer (NJYSA) | NJ | Yes | Large | **Needs lookup** | NJ — heavy representation in target demo |
| Washington Youth Soccer (WSYSA) | WA | Yes | Medium | **Needs lookup** | GotSport per research |
| Colorado Youth Soccer (CYSA) | CO | Yes | Medium | **Needs lookup** | GotSport per research |
| Pennsylvania West | PA-W | Yes (likely) | Medium | **Needs lookup** | USYS affiliate; PA East/West split |
| Pennsylvania East (EPYSA) | PA-E | Yes (likely) | Medium | **Needs lookup** | Check separately from PA-W |

### Skip States

| State Association | Platform | Reason to Skip |
|-------------------|----------|----------------|
| Illinois (IYSA) | Affinity Soccer | No Affinity scraper — deferred post-DSS per USYS research |
| California (CCSL/Cal North) | GotSport + SincSports | Complex split — do TX/FL/NY/NJ first |

### How to Find Each Org ID

**Method 1 — Browser inspection (no auth required):**
Navigate to `gotsport.com` → find the state association's event listing → inspect the URL. Example: `https://system.gotsport.com/org_event/org_events?org_id=12345&page=1` reveals the org_id.

**Method 2 — GotSport API test:**
```bash
# Test a known org ID to confirm the endpoint works
curl "https://system.gotsport.com/api/v1/team_ranking_data?org_id=REPLACE_WITH_ID" \
  -H "Accept: application/json" | jq '.[] | .team_id' | head -20
```

**Method 3 — DB cross-reference (fastest if games already exist):**
```sql
-- Find org IDs already embedded in game data for state cup events
SELECT 
  event_name,
  source_id,
  COUNT(*) as game_count
FROM games
WHERE is_hidden = false
  AND (
    event_name ILIKE '%state cup%'
    OR event_name ILIKE '%STYSA%'
    OR event_name ILIKE '%ENYYSA%'
    OR event_name ILIKE '%NJYSA%'
    OR event_name ILIKE '%WSYSA%'
  )
GROUP BY event_name, source_id
ORDER BY game_count DESC
LIMIT 50;
-- source_id format is gs_api_{eventId}_{matchId}
-- eventId may encode the org_id — inspect the pattern
```

---

## Step 2: Current Team Discovery Configuration

**Where team IDs are seeded (to be verified by Presten):**

Based on the GotSport API Scraper note and the Daily Pipeline Cadence Scope doc:
- `crawl-gotsport-api-parallel.js` either reads from `GSAPI_TEAM_IDS` env var OR calls `/api/v1/team_ranking_data` to discover all ranked team IDs
- `GSAPI_SKIP_RANKINGS=true` skips the rankings endpoint discovery
- The daily pipeline (`run-daily.sh`) generates `active_team_ids.txt` from the games table

**What to check:**
```bash
# Look for where team IDs enter the pipeline
grep -n "team_ranking_data\|GSAPI_TEAM_IDS\|org_id" crawl-gotsport-api-parallel.js

# Look for any existing org ID config
ls config/
cat config/gotsport-orgs.json 2>/dev/null || echo "File not found"
```

---

## Step 3: Implementation Plan

### Option A — API Endpoint for Org-Level Discovery

If GotSport supports `/api/v1/team_ranking_data?org_id={id}`:

```bash
# scripts/discover-teams-by-org.sh
#!/bin/bash
# Discovers team IDs for a list of GotSport org IDs
# Usage: bash discover-teams-by-org.sh | tee new_org_team_ids.txt

ORG_IDS=(
  "STYSA_ORG_ID"    # TX — replace with actual
  "FL_ORG_ID"       # FL
  "ENYYSA_ORG_ID"   # NY
  "NJYSA_ORG_ID"    # NJ
  "WSYSA_ORG_ID"    # WA
  "CYSA_ORG_ID"     # CO
)

for ORG_ID in "${ORG_IDS[@]}"; do
  echo "Fetching org $ORG_ID..." >&2
  curl -s "https://system.gotsport.com/api/v1/team_ranking_data?org_id=${ORG_ID}" \
    -H "Accept: application/json" \
    | jq -r '.[].team_id' 2>/dev/null
  sleep 1200  # respect rate limit
done
```

### Option B — Add Org IDs to Existing Config File

If there's a `config/gotsport-orgs.json` or similar:

```json
{
  "org_ids": [
    {"id": "STYSA_ID", "name": "Texas Soccer (STYSA)", "state": "TX"},
    {"id": "FL_ID", "name": "Florida Youth Soccer", "state": "FL"},
    {"id": "ENYYSA_ID", "name": "Eastern NY Youth Soccer", "state": "NY"},
    {"id": "NJYSA_ID", "name": "NJ Youth Soccer", "state": "NJ"},
    {"id": "WSYSA_ID", "name": "Washington Youth Soccer", "state": "WA"},
    {"id": "CYSA_ID", "name": "Colorado Youth Soccer", "state": "CO"}
  ]
}
```

### Integration Point

The org-level discovery should run BEFORE the ranked team ID fetch in the daily pipeline, so these IDs are included in every sync:

```bash
# In run-daily.sh (proposed addition):
# Step 0 (before active ID query):
echo "Discovering org-level team IDs..."
GSAPI_SKIP_RANKINGS=false bash discover-teams-by-org.sh >> /tmp/active_team_ids.txt

# Or: add org-discovered IDs to the DB's teams table first,
# then they'll naturally appear in the active ID query
```

---

## Step 4: Test Run — TX State Association

**Before adding all org IDs:** Test on TX alone.

### Pre-Test Query — Establish Baseline

```sql
-- How many TX state cup teams do we already have?
SELECT COUNT(DISTINCT unnest(ARRAY[home_team_id, away_team_id])) as team_count
FROM games
WHERE is_hidden = false
  AND (event_name ILIKE '%STYSA%' OR event_name ILIKE '%Texas%State%Cup%');

-- What's the current total team count?
SELECT COUNT(*) FROM teams;
```

### Discovery Test

```bash
# Run org discovery for TX only
curl -s "https://system.gotsport.com/api/v1/team_ranking_data?org_id=TX_ORG_ID" \
  -H "Accept: application/json" | jq '.[].team_id' | wc -l
# Expected: hundreds to a few thousand team IDs
```

### New Teams Discovered

```sql
-- After inserting discovered team IDs:
-- Count teams NOT already in our DB
SELECT COUNT(*) FROM (
  SELECT unnest(ARRAY[TX_DISCOVERED_ID_LIST]) AS gotsport_id
  EXCEPT
  SELECT gotsport_id FROM teams WHERE gotsport_id IS NOT NULL
) new_teams;
```

### Spot Check (5 Random New Teams)

For each of 5 randomly selected new team IDs:
- Confirm they are real USYS/state-affiliated teams (not test accounts)
- Confirm they are not already in the DB under a different ID (dedup check)
- Confirm their matches endpoint returns real game data

---

## Step 5: Expected Results and Success Criteria

| Metric | Target | Pass/Fail |
|--------|--------|-----------|
| Org IDs identified | ≥ 5 confirmed | — |
| TX test discovery returns team IDs | > 100 new team IDs | — |
| New team IDs not already in DB | > 50% of discovered | — |
| Org-level API endpoint responds without auth | Yes | Required |
| Estimated extrapolated game count (all orgs) | 20,000–35,000 | Validate vs research estimate |
| Dedup conflicts on test crawl | < 5% overlap with existing records | — |

---

## Estimated Impact — Full Deployment

| State | Est. New Teams | Est. New Games | Notes |
|-------|---------------|----------------|-------|
| TX (STYSA) | 500–2,000 | 8,000–15,000 | Largest state; some already in DB |
| FL | 300–1,000 | 4,000–8,000 | Significant state cup volume |
| NY (ENYYSA) | 200–800 | 3,000–6,000 | High elite team density |
| NJ (NJYSA) | 150–600 | 2,000–5,000 | Target demo for DSS |
| WA + CO + PA | 200–700 | 3,000–8,000 | Medium states |
| **Total** | **1,350–5,100** | **20,000–42,000** | Aligns with USYS research estimate |

---

## DB Queries to Run After Production Deployment

```sql
-- Confirm new game count after full org expansion
SELECT 
  source,
  COUNT(*) as total_games,
  COUNT(*) FILTER (WHERE created_at > NOW() - INTERVAL '2 days') as new_games
FROM games
WHERE is_hidden = false
  AND source IN ('gotsport_api', 'gotsport')
GROUP BY source;

-- Check if state cup events are now better covered
SELECT 
  COUNT(*) FILTER (WHERE event_name ILIKE '%state cup%') as state_cup_games,
  COUNT(*) FILTER (WHERE event_name ILIKE '%state league%') as state_league_games
FROM games
WHERE is_hidden = false;
```

---

## Files to Create

| File | Purpose |
|------|---------|
| `scripts/discover-teams-by-org.js` (or `.sh`) | New discovery script for org-level team enumeration |
| `config/gotsport-orgs.json` | Org ID list (or add to existing config structure) |
| `Reports/gotsport-org-expansion-test-2026-04-24.md` | This document |

---

## Blocker Note

This report represents the research and planning portion of the task. **Steps 1–5 require:**
1. DB access (`psql youth_soccer`) to run baseline queries
2. Ability to browse/curl GotSport to find org IDs
3. Ability to modify config files and run discovery scripts

Presten should execute Steps 1 and 4 first (baseline query + TX test), then expand to all 6 org IDs.

---

## References

- `Data Sources/USYS State Cup Source Research.md` — primary source for this task; Section 1 (Platform Map) and Section 3 (Top 3 Sources)
- [[GotSport API Scraper]] — discovery endpoint + GSAPI_TEAM_IDS env var
- `Product & Planning/Daily Pipeline Cadence Scope.md` — `get-active-team-ids.sh` that org-discovered teams should feed into

---

*FORGE — 2026-04-24*
