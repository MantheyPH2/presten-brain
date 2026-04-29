---
type: pipeline-baseline
title: May 1 — Official Source Baseline
date_established: 2026-05-01
pipeline_run: first-production-run
stage: stage-1
stage2_authorization_evidence: true
status: pre-filled (actuals pending pipeline run)
author: FORGE
created: 2026-04-29
tags: [infrastructure, pipeline, stage1, baseline, stage2, may1, forge]
---

# May 1 — Official Source Baseline

> **Purpose:** Authoritative record of Stage 1 forecast vs. actual game volumes. Section 2 actuals are filled by FORGE within 2 hours of the May 1 pipeline run. Section 5 is filled by SENTINEL after reviewing actuals. This document is a required input for the Stage 2 Expansion Authorization Trigger Document (Section 1).

**Prepared by:** FORGE — 2026-04-29  
**Actuals status:** PENDING — pipeline has not yet run  
**SENTINEL review:** PENDING actuals fill

---

## Section 1 — Header

```
Pipeline stage:          Stage 1 (TX pilot — TYSA)
First run date:          May 1, 2026 (target)
Pipeline version:        Current (gotsport_api + tgs_athleteone + tgs + tgs_rl + gotsport_html)
Stage 1 org-IDs:         TYSA (Texas Youth Soccer Association) — org-ID pending browser confirmation
Forecast source:         Infrastructure/May 1 — Per-Source Game Volume Forecast.md (2026-04-27)
Stage 2 auth evidence:   This document is submitted to Stage 2 Trigger Document, Section 1 on May 4
```

---

## Section 2 — Per-Source Forecast vs. Actual Table

Forecast values from `Infrastructure/May 1 — Per-Source Game Volume Forecast.md`. Actuals filled by FORGE post-run.

> **Note:** The forecast covers *incremental new games ingested on the first daily run* (not cumulative DB total). May 1 = Thursday (weekday); weekend runs (May 3–4) will show higher volumes.

| Source | Org IDs in Config | Forecast Games (first daily run) | Actual Games (May 1 run) | Delta | Status |
|--------|------------------|----------------------------------|--------------------------|-------|--------|
| gotsport_api | ~107,528 ranked team IDs (no new org-IDs until TYSA confirmed) | 300–1,200 | [FORGE fills post-run] | — | — |
| tgs_athleteone | ECNL Girls, ECNL Boys (TGS direct) | 80–300 | [FORGE fills post-run] | — | — |
| tgs | TGS main (older ECNL records) | 20–80 | [FORGE fills post-run] | — | — |
| tgs_rl | ECNL RL via TGS feed | 20–80 | [FORGE fills post-run] | — | — |
| gotsport_html | HTML scraper (supplementary) | 10–50 | [FORGE fills post-run] | — | — |
| **TYSA (gotsport_api Stage 1)** | **[OPEN — pending Presten org-ID]** | **5,000–25,000 (if Stage 1 run completes; one-time discovery)** | **[FORGE fills post-run]** | — | **OPEN — blocked on org-ID** |

**Overall Stage 1 forecast total (excluding TYSA):** 430–1,710 incremental games  
**Overall Stage 1 forecast total (including TYSA if org-ID confirmed before run):** 5,430–26,710 games  

**Stage 2 authorization threshold:** Stage 1 must achieve ≥ GREEN status on all active sources, with no critical source failures, before SENTINEL authorizes Stage 2 expansion.

---

## Section 3 — Source Health Flags

Pre-populated known risks and status items:

| Item | Status | Notes |
|------|--------|-------|
| TYSA org-ID | **OPEN — pending Presten browser session** | Only actual launch blocker. Without org-ID, Stage 1 TX crawl cannot run. Pipeline runs without TYSA; TYSA adds Stage 1 volume when org-ID confirmed. |
| FM1 behavior (ECNL classifier) | **Pre-draft staged, code search pending** | Presten runs grep commands from `Infrastructure/FM1-FM2 Audit — Presten Execution Card.md`; FORGE closes audits in < 30 min. Does NOT block May 1 pipeline launch. |
| FM2 behavior (GA ASPIRE classifier) | **Pre-draft staged, code search pending** | Same as FM1. Non-blocking for May 1. |
| Archive step (Step 5.5) | **NOT running May 1** | Archive step (`ARCHIVE_STEP=true`) is NOT activated for May 1. Requires Presten dry-run authorization first. GROUP BY bug fixed in spec; Presten must apply fix to `archive-inactive-teams.js` before running. |
| ECNL migration (Option 1) | **No FORGE action required on May 1** | ECNL migration is a June 1 event. Option 1 (No Re-tag) means zero pipeline changes on May 1. FORGE monitors ECNL ingestion continuity post-June 1. |
| tgs_rl staleness | **Staleness check pending before May 9** | ECNL RL via TGS feed — ingestion recency unverified. Verify before classifying a May 1 RED result for tgs_rl as pipeline failure. |

---

## Section 4 — Post-Run Completion Protocol

FORGE fills actuals within 2 hours of May 1 pipeline run completing:

1. **Run per-source game count query**
   ```sql
   SELECT source, COUNT(*) AS new_games, MAX(ingested_at) AS last_ingested
   FROM games
   WHERE ingested_at > '[May 1 pipeline start timestamp]'
   GROUP BY source ORDER BY source;
   ```

2. **Fill "Actual Games" column** in Section 2 for each source.

3. **For TYSA (if org-ID confirmed and Stage 1 run executed):**
   ```sql
   SELECT source_org_id, COUNT(*) AS games_ingested
   FROM games
   WHERE source = 'gotsport'
     AND ingested_at >= NOW() - INTERVAL '24 hours'
   GROUP BY source_org_id ORDER BY games_ingested DESC;
   ```

4. **Calculate delta** (actual vs. forecast %) for each source.

5. **Set status** for each source:
   - **PASS (GREEN):** Actual ≥ GREEN floor from forecast document
   - **YELLOW:** Actual between RED ceiling and GREEN floor — investigate before authorizing Stage 2
   - **FAILED (RED):** Actual below RED ceiling — diagnose before proceeding
   - **OPEN:** Source not yet configured (TYSA if org-ID not received)

6. **Set overall Stage 1 status:**
   - **CONFIRMED:** All active sources GREEN or YELLOW, no critical failures
   - **NEEDS-REVIEW:** One or more sources RED; specific anomaly documented
   - **FAILED:** Pipeline did not run or gotsport_api is RED with no backoff explanation

7. **Update `Infrastructure/Pipeline Week 1 Monitoring Log.md`** with first-run data.

8. **Notify SENTINEL** in next briefing: "Stage 1 first-run reconciliation complete. Status: [CONFIRMED/NEEDS-REVIEW/FAILED]. Details in May 1 Official Source Baseline, Section 4."

9. **Update Stage 2 Trigger Document Section 1** with actuals and status determination.

---

## Section 5 — Stage 2 Authorization Decision

*SENTINEL fills this section after reviewing Section 4 actuals.*

```
Stage 1 actual result:   [ ] CONFIRMED  [ ] NEEDS-REVIEW  [ ] FAILED

Stage 2 expansion:       [ ] AUTHORIZED  [ ] HOLD  [ ] ESCALATE TO PRESTEN

Conditions for Stage 2:
_______________________________________________________________

SENTINEL decision date:  ___________
SENTINEL signature:      ___________
```

---

## References

- `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` — forecast source
- `Infrastructure/Pipeline Week 1 Monitoring Log.md` — running daily actuals log
- `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` — org-ID manifest
- `Infrastructure/Stage 2 Expansion Authorization Trigger Document.md` — Section 1 receives actuals from this doc on May 4
- `Infrastructure/May 1 — Stage 1 First-Run Reconciliation Report.md` — detailed per-source reconciliation (filed within 24 hrs of run)
- `Infrastructure/May 1 Stage 1 — Launch Confidence Brief.md` — pre-launch readiness assessment

*FORGE — 2026-04-29*
