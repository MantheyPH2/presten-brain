---
title: Non-GotSport Source #2 — USL Academy Implementation Build Plan
tags: [infrastructure, sources, usl-academy, gotsport, pipeline, implementation, dss, evo-draw]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: filed — org-ID pending browser session; activate on EDP Gate 1 + Gate 2 pass
task: task-2026-04-28-usl-academy-source2-build-plan
due: 2026-05-06
---

# Non-GotSport Source #2 — USL Academy Implementation Build Plan

> [!info] Purpose
> Day-by-day execution blueprint for implementing USL Academy, ranked #2 in the Non-GotSport Source Implementation Priority List. USL Academy uses the GotSport API (identical to existing pipeline). Implementation path is a config add — same mechanic as EDP (Source #1). Activation gates off EDP Gate 1 (parse quality) + Gate 2 (ranking impact), expected ~May 4–6.

**Prepared by:** FORGE — 2026-04-28  
**Input document:** `Infrastructure/Non-GotSport Source Implementation Priority List.md`  
**Research source:** `Data Sources/USL-Academy-OrgID-Research-2026-04-27.md`  
**DSS deadline:** May 16  
**Implementation window:** EDP Gate 2 pass day (~May 4–6) → same-session config add

---

## Section 1: Source Identity and Summary

| Field | Value |
|-------|-------|
| Source name | **USL Academy** |
| Platform / data format | GotSport API — identical to existing `crawl-gotsport-api-parallel.js` pipeline |
| Parser reuse | **YES** — Full reuse of existing GotSport API pipeline. No new parser code required. |
| Existing parser | `crawl-gotsport-api-parallel.js` (project root) |
| League scope | Boys U13–U19; professional-pathway clubs affiliated with USL Championship / USL League One |
| Clubs | ~90 clubs (national) |
| Game volume estimate | 2,000–5,000 games/year (net new after dedup with existing ranked-team discovery data) |
| Geographic coverage | National — professional pathway clubs across all U.S. regions |
| Pre-DSS Safe | **YES** — same config-add path as EDP; zero engineering risk |
| Blocker | USL Academy org-ID not yet confirmed — pending Presten browser session at `system.gotsport.com` |
| Activation trigger | EDP Gate 1 (parse quality) + Gate 2 (ELO ranking impact) both confirmed passing |
| ELO coordination required | **YES** — boys ratings impact flag if any team shifts > 50 pts from USL Academy data |

> [!note] This is a config add, not a build
> USL Academy uses the GotSport API at the org level — same path as all other GotSport sources. Implementation requires one config line, a syntax check, and post-run validation. No parser development.

> [!warning] Boys calibration context
> ELO is in active boys Option A calibration evaluation (April 29–May 5 window). USL Academy data may affect boys ratings. ELO must be notified before USL Academy crawl runs and must confirm calibration-safe within 24 hours of ingestion. Do not proceed to Gate 3 without ELO confirmation.

---

## Section 2: Org-ID Inventory

Org-ID confirmation is pending the browser session at `system.gotsport.com`. USL Academy is in `GotSport Org-ID Master Reference — April 2026.md` Section 2, Row 1 (`pending-browser-lookup`).

| Org-ID | Organization | Verified? | Source of Verification |
|--------|-------------|-----------|------------------------|
| [TBD-ORG-ID] | USL Academy (national) | **NO — pending browser session** | Browser lookup at `system.gotsport.com` → search "USL Academy" or "United Soccer League Academy" |

**GotSport presence confidence:** High (85–90%). USL Soccer is a confirmed GotSport organization; a national USL Academy org-ID almost certainly exists. If browser session confirms "not on GotSport": escalate to SENTINEL — USL Academy becomes a non-GotSport platform research task.

**Pre-config baseline query** (run before activating org-ID to establish existing coverage):
```sql
-- Existing USL Academy game count via ranked-team discovery
SELECT COUNT(*) AS existing_usl_academy_games, MAX(game_date) AS latest_game
FROM games
WHERE event_name ILIKE '%USL Academy%'
   OR event_name ILIKE '%United Soccer League Academy%'
   OR event_tier = 'usl_academy';
```
Record this count before config add. Delta after first crawl = net new games attributable to org-ID addition.

---

## Section 3: Day-by-Day Build Timeline

> [!note] Timeline is relative to EDP Gate 2 pass day
> "Day 0" = the day EDP Gate 2 is confirmed passing. USL Academy activates in the same session as EDP Gate 2 confirmation if EDP add runs cleanly. Specific calendar dates depend on EDP Gate 2 confirmation date (estimated May 4–6).

| Step | Expected Date | Task | Owner | Success Criteria |
|------|--------------|------|-------|-----------------|
| 0 | EDP Gate 2 pass day (~May 4–6) | Receive USL Academy org-ID from browser session results. Open `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` → find USL Academy entry → replace `[TBD-ORG-ID]` → paste into config. Verify syntax. Update Org-ID Master Reference. Notify ELO: "USL Academy adding — run post-ingestion boys ranking comparison." | FORGE | `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"` passes. ELO notified before next crawl runs. |
| 1 | Day 0 + 1 day | First pipeline run with USL Academy org-ID active. Run Q1 (game count by org-ID), Q2 (recency check), Q3 (dedup check). Compare against pre-config baseline. | FORGE + Presten | Games ingested > 0 from USL Academy org-ID. No null org_id rows. Most recent game_date ≥ 2025. |
| 2 | Day 0 + 2 days | Review game count vs. expected (2,000–5,000/year annualized; prorated to ~1,000–2,500 for spring season). Check event_tier classification. Spot-check 5–10 game records for field completeness. **ELO runs post-ingestion boys ranking comparison.** | FORGE + ELO | Game count within expected range. event_tier populated correctly. Parse error rate < 5%. ELO confirmation filed: "USL Academy data is calibration-safe." |
| 2 | Same day | FORGE files first-run validation report to `Infrastructure/Source #2 — USL Academy First Run Validation.md`. | FORGE | Validation report filed. SENTINEL notified. |
| 3 | Day 0 + 3 days | SENTINEL reviews validation report and issues Stage 2 authorization for USL Academy to remain in live daily pipeline. Presten confirms USL Academy is live in production. | SENTINEL + Presten | SENTINEL Gate 3 signed. USL Academy confirmed active. |

**Compressed timeline justification:** Same as EDP — config add only, no parser work. Standard 6-day template compresses to 3 days because Days 0–1 are config add + first run, not a development phase.

---

## Section 4: ELO Coordination Requirements

> [!important] Mandatory — do not skip
> USL Academy boys data may affect boys ratings during active Option A calibration evaluation. ELO must confirm calibration-safe before Gate 3 is issued.

**Specific coordination protocol:**

1. **Before config add (Day 0):** FORGE files SENTINEL queue item: "USL Academy org-ID confirmed — adding to config. ELO: please run post-ingestion boys ranking comparison after next pipeline run. Threshold: any boys team shift > 50 pts from USL Academy data alone = anomaly flag."

2. **After ingestion (Day 2):** ELO runs post-ingestion boys ranking comparison:
   - Compare boys rankings before and after USL Academy games are factored in
   - Flag any team moving > 50 pts from USL Academy data alone
   - Confirm girls ranking impact acceptable (< 30 pts avg shift)

3. **ELO confirmation (Day 2, within 24 hours of ingestion):** ELO files: "USL Academy data is calibration-safe. Boys rating shifts within threshold. Girls impact acceptable."

4. **If ELO flags anomaly:**
   - FORGE immediately disables USL Academy in config (remove org-ID line; verify syntax)
   - FORGE files `Infrastructure/Source #2 — USL Academy First Run Failure Report.md` within 2 hours
   - FORGE files SENTINEL queue item (priority: urgent) with anomaly details
   - Do not re-activate without SENTINEL + ELO joint review

---

## Section 5: Validation Gates

Three gates must pass before USL Academy implementation is called complete.

### Gate 1 — Parse Quality

```sql
-- Q1: USL Academy game count by org-ID (run immediately after first pipeline run)
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
- Games ingested > 0 from USL Academy org-ID
- Game count within ±30% of expected range from Section 1

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

### Gate 2 — Ranking Impact (ELO)

- ELO runs post-ingestion boys ranking comparison (before and after USL Academy data active)
- No boys team moves > 50 pts from USL Academy data alone (DSS-window threshold; tighter than EDP because of active calibration)
- Girls ranking impact acceptable (< 30 pts avg shift)
- ELO files confirmation: "Source #2 USL Academy data is calibration-safe" before Gate 3 can be issued

> [!note] EDP vs. USL Academy threshold
> EDP boys threshold is > 200 pts (Gate 2). USL Academy boys threshold is > 50 pts — tighter because USL Academy teams are specifically in the professional-pathway calibration band where active Option A evaluation is occurring. Flag to ELO if boys Option A evaluation is still in progress at the time of USL Academy ingestion.

### Gate 3 — SENTINEL Authorization

- FORGE files first-run validation report at `Infrastructure/Source #2 — USL Academy First Run Validation.md`
- SENTINEL reviews and issues production authorization
- Presten confirms USL Academy is active in production daily pipeline

All three gates must pass before FORGE files the validation report confirming USL Academy implementation complete.

---

## Section 6: Rollback Plan

If Gate 1 or Gate 2 fails:

1. Remove USL Academy org-ID from `crawl-gotsport-api-parallel.js` (do not delete ingested data)
   - Verify syntax: `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"`
2. FORGE files failure report: `Infrastructure/Source #2 — USL Academy First Run Failure Report.md` within 2 hours of failure classification
3. File SENTINEL queue item (priority: high) with failure details
4. ELO notified immediately if Gate 2 (boys rating anomaly) is the failure cause
5. FORGE does NOT attempt re-activation without SENTINEL review of the failure report

If Gate 2 fails (ELO flags boys rating anomaly):
- ELO and SENTINEL jointly review before any re-activation
- If anomaly is from USL Academy data quality (bad game records): FORGE files data quality report, waits for SENTINEL direction on whether to re-ingest with corrections
- If anomaly is from calibration interaction: SENTINEL defers USL Academy to post-Option-A-calibration-window (post May 5)

---

## References

- `Infrastructure/Non-GotSport Source #1 — Implementation Build Plan.md` — EDP; structural template for this document
- `Data Sources/USL-Academy-OrgID-Research-2026-04-27.md` — org-ID research and GotSport presence assessment
- `Infrastructure/Non-GotSport Source Implementation Priority List.md` — confirms USL Academy = Source #2
- `Infrastructure/EDP + USL Academy — Config Activation Card.md` — rapid execution card (already filed)
- `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` — pre-written USL Academy config entry
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — USL Academy in Section 2 (browser session pending)
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config add procedure and entry format

*FORGE — 2026-04-28*
