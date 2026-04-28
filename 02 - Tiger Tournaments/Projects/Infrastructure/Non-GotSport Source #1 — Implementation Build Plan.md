---
title: Non-GotSport Source #1 — Implementation Build Plan
tags: [infrastructure, sources, edp, gotsport, pipeline, implementation, dss, evo-draw]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: filed
task: task-2026-04-28-source1-non-gotsport-build-plan
due: 2026-05-02
---

# Non-GotSport Source #1 — Implementation Build Plan

> [!info] Purpose
> Day-by-day execution blueprint for implementing EDP Soccer (Eastern Development Program), ranked #1 in the Non-GotSport Source Implementation Priority List. Because EDP uses the GotSport API (identical to the existing pipeline), implementation is a config add — not a parser build. The timeline is compressed accordingly.

**Prepared by:** FORGE — 2026-04-28  
**Input document:** `Infrastructure/Non-GotSport Source Implementation Priority List.md`  
**DSS deadline:** May 16  
**Implementation window:** Within 1 hour of browser session org_id confirmation (target: April 28–29)

---

## Section 1: Source Identity and Summary

| Field | Value |
|-------|-------|
| Source name | **EDP Soccer (Eastern Development Program)** |
| Platform / data format | GotSport API — identical to existing `crawl-gotsport-api-parallel.js` pipeline |
| Parser reuse | **YES** — Full reuse of existing GotSport API pipeline. No new parser code required. |
| Existing parser | `crawl-gotsport-api-parallel.js` (project root) |
| Game volume estimate | 3,000–8,000 games/year |
| Geographic coverage | Northeast U.S. — NJ, NY, PA, CT, MA, MD, RI, DE, VA, VT |
| Clubs | ~150+ clubs, U8–U19 |
| Pre-DSS Safe | **YES** |
| Blocker | EDP org_id not yet confirmed — pending Presten browser session at `system.gotsport.com` |
| Implementation window | Within 1 hour of org_id confirmation → first games on next pipeline run |

> [!note] This is a config add, not a build
> EDP uses the GotSport API at the org level — the same path as every other GotSport source in the pipeline. Implementation requires one config line, a syntax check, and validation after the next run. There is no parser development phase.

---

## Section 2: Parser Architecture Decision

**Selection: Option A — Full reuse of existing parser**

EDP is on GotSport. The existing `crawl-gotsport-api-parallel.js` scraper already handles GotSport org-level API requests, pagination, field extraction, team name normalization, and database insertion. Adding EDP requires:

1. One config entry (org_id value in the org-ID array)
2. Syntax verification (`node -c crawl-gotsport-api-parallel.js`)
3. No code changes to the parser, ingestion logic, or DB schema

**Pre-flight dedup note:** EDP has ~150+ clubs in the Northeast. Some EDP games may already be in the DB via ranked-team discovery. Run the pre-flight query below before activating to establish a baseline. Expected: some overlap; the org-level crawl fills unranked divisions not yet captured.

```sql
-- Pre-config baseline: existing EDP game count
SELECT COUNT(*) FROM games
WHERE event_name ILIKE '%EDP%' OR event_name ILIKE '%Eastern Development%';
```

If result > 0: note the count. This is the pre-config baseline. Delta after first run = net new games.

---

## Section 3: Day-by-Day Build Timeline

> [!note] Timeline assumes browser session org_id confirmation arrives April 28 or 29

| Day | Date | Task | Owner | Success Criteria |
|-----|------|------|-------|-----------------|
| 0 | April 28–29 | Receive EDP org_id from browser session. Open `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` → replace `[TBD-ORG-ID]` with confirmed org_id. Paste into config. Verify syntax. Update Org-ID Master Reference. | FORGE | `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"` passes. Master Reference Section 2 updated to `confirmed-[org_id]`. |
| 1 | April 29–30 | First pipeline run with EDP org_id active. Run Q1 (game count by org_id), Q2 (recency check), Q3 (source error check). Compare against pre-config baseline. | FORGE + Presten | New games ingested > 0. No null org_id rows. Most recent game_date ≥ 2025. |
| 2 | April 30 – May 1 | Review game count vs. expected (3,000–8,000/year annualized; prorated to ~1,500–4,000 for April). Check event_tier classification assigns correct tier to EDP games. Spot-check 5–10 game records for field completeness. | FORGE | Game count within expected range. `event_tier` populated correctly for EDP events. Parse error rate < 5%. |
| 3 | May 1–2 | ELO runs post-ingestion ranking comparison. Confirm no anomalous rating shifts > 200 pts from EDP data alone. ELO files calibration-safe confirmation. | Presten + ELO | ELO confirmation filed: "EDP data is calibration-safe." Avg rating shift from EDP games < 30 pts. |
| 4 | May 2 | FORGE files first-run validation report to `Infrastructure/EDP Soccer — First Run Validation Report.md`. Update Stage 2 Authorization Gate if EDP config add is one of the confirmed new sources. | FORGE | Validation report filed. SENTINEL notified. |

**Compressed timeline justification:** EDP uses the existing GotSport API pipeline. There is no parser scaffolding, test fixture extraction, or DB schema work. The standard 6-day build plan compresses to 4 days because Days 0–1 are a config add + first run, not a development phase.

---

## Section 4: Integration Requirements

- [x] **Config file update:** Add EDP org_id to `crawl-gotsport-api-parallel.js` under `// EDP Soccer` section (pre-written entry in `Stage 2 Config Pre-Population` document)
- [ ] **Cron schedule:** No change — EDP runs on the existing daily pipeline schedule. No separate schedule needed.
- [ ] **Dedup rules:** No new dedup logic required. Existing `(source, source_id)` dedup key handles GotSport game IDs natively. Some EDP games already in DB will resolve as duplicates automatically.
- [ ] **`ecnl_verified` flag:** Not applicable to EDP. EDP is a regional Tier 2 league, not ECNL. No classifier update needed.
- [ ] **`event_tier` classification:** Review FORGE's event tier parsing rules to confirm EDP events classify correctly. EDP is approximately Tier 2 (regional competitive). If EDP events are miscategorized, update the tier parsing ruleset — flag to SENTINEL if a code change is needed.

**Net code changes required:** Zero. Config add only.

---

## Section 5: Validation Gates

Three gates must pass before calling EDP implementation complete.

### Gate 1 — Parse Quality

```sql
-- Q1: EDP game count by org_id (run immediately after first pipeline run with EDP active)
SELECT source_org_id, COUNT(*) AS games_ingested
FROM games
WHERE source = 'gotsport'
  AND created_at > NOW() - INTERVAL '24 hours'
GROUP BY source_org_id
ORDER BY games_ingested DESC;
```

- Parse error rate < 5%
- All required fields populated: `team_name`, `opponent_name`, `date`, `score`, `league_name`
- No duplicate game rows (`(source, source_id)` dedup check passes)
- Games ingested > 0 from EDP org_id

```sql
-- Q2: Dedup check — no duplicate games from this crawl
SELECT source_id, COUNT(*) AS count
FROM games
WHERE source = 'gotsport'
  AND created_at > NOW() - INTERVAL '24 hours'
GROUP BY source_id
HAVING COUNT(*) > 1;
```

Expected: 0 rows.

### Gate 2 — Ranking Impact

- ELO runs post-ingestion ranking comparison (before and after EDP data active)
- Avg rating shift from EDP games: < 30 pts (expected with initial dataset)
- No team moves more than 200 pts from EDP source data alone
- ELO files confirmation: "EDP data is calibration-safe" before Stage 2 expansion proceeds

### Gate 3 — SENTINEL Authorization

- FORGE files first-run validation report
- SENTINEL reviews and issues production authorization for EDP to remain in live daily pipeline
- Presten confirms EDP is active in production

All three gates must pass before FORGE files May 2 briefing confirming EDP implementation complete.

---

## Section 6: Rollback Plan

If Gate 1 or Gate 2 fails:

1. Remove EDP org_id from `crawl-gotsport-api-parallel.js` (do not delete ingested data)
2. Verify syntax: `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"`
3. FORGE files failure report: `Infrastructure/EDP Soccer — First Run Failure Report.md` within 2 hours of failure classification
4. File SENTINEL queue item (priority: high) with failure details
5. FORGE does NOT attempt a second activation without SENTINEL review of the failure report

**FORGE does not proceed to USL Academy (Source #2) until EDP Gate 1 and Gate 2 are confirmed passing.**

---

## References

- `Infrastructure/Non-GotSport Source Implementation Priority List.md` — priority ranking and EDP summary
- `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` — pre-written EDP config entry
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — EDP org_id tracking (Section 2)
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config add procedure
- `Infrastructure/EDP + USL Academy — Config Activation Card.md` — rapid execution card for config add
- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 gate conditions
- `Infrastructure/Non-GotSport Source Implementation Priority List.md` Section 3 — Source 2 (USL Academy) follows after EDP Gate 1 + Gate 2 pass

*FORGE — 2026-04-28*
