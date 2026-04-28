---
title: TYSA Config Add — Execution Checklist
tags: [infrastructure, gotsport, tysa, config, stage1, pipeline, execution, forge]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: ready — execute within 30 minutes of receiving TYSA org-ID from browser session
task: task-2026-04-28-tysa-config-add-execution-checklist
due: 2026-04-29
---

# TYSA Config Add — Execution Checklist

> [!info] Purpose
> Step-by-step execution reference for FORGE to complete the TYSA org-ID config add within 30 minutes of receiving the org-ID from Presten's browser session. Completing this checklist signs the May 1 Stage 1 Config Final Review (Section 5) and fully authorizes May 1 deployment.

**Config file:** `crawl-gotsport-api-parallel.js` (project root)  
**Config entry format:** See `Infrastructure/GotSport Org-ID Config Edit Reference.md` Section 2 (USYS State Association template)  
**Time target:** All steps complete within 30 minutes of org-ID receipt

---

## Section 1: Pre-Receipt Readiness (Complete Now — Before Browser Session)

FORGE confirms these are true before the browser session begins. If any item is NOT READY, resolve it before Presten runs the browser session.

- [ ] **Config file location confirmed:** `crawl-gotsport-api-parallel.js` in project root — verify with `ls *.js | grep crawl-gotsport`
- [ ] **Config entry format documented** — USYS State Association template from Config Edit Reference:
  ```javascript
  // Texas Youth Soccer Association (USYS TX) — added [YYYY-MM-DD] by Presten
  [TYSA_ORG_ID], // TYSA — Texas statewide
  ```
- [ ] **Syntax check command ready:**
  ```bash
  node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"
  ```
- [ ] **Duplicate check command ready:**
  ```bash
  grep -n "[TYSA_ORG_ID]" crawl-gotsport-api-parallel.js
  ```
  Expected after add: exactly 1 occurrence. If 2+: duplicate — remove one before proceeding.
- [ ] **Pre-config baseline query ready** (run before adding org-ID):
  ```sql
  SELECT COUNT(*) AS existing_tysa_games
  FROM games
  WHERE event_name ILIKE '%TYSA%'
     OR event_name ILIKE '%Texas Youth Soccer%'
     OR source_org_id = '[TYSA_ORG_ID_PLACEHOLDER]';
  ```
  Record count. Delta after first run = net new TYSA games.
- [ ] **First-run monitoring query ready** (run after first post-May-1 pipeline run):
  ```sql
  SELECT 
    COUNT(*) AS total_games,
    MIN(game_date) AS earliest_game,
    MAX(game_date) AS latest_game
  FROM games
  WHERE source_org_id = '[TYSA_ORG_ID]';
  ```
- [ ] **May 1 Stage 1 Config Final Review** open and ready for Section 5 signature:
  `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md`

**Pre-receipt status:** READY / NOT READY

---

## Section 2: Execution Steps (Run Within 30 Minutes of Org-ID Receipt)

Execute in order. Do not skip steps.

---

### Step 1 — Run Pre-Config Baseline Count (< 2 min)

Before touching the config file, record the existing TYSA game count:

```sql
SELECT COUNT(*) AS existing_tysa_games
FROM games
WHERE event_name ILIKE '%TYSA%'
   OR event_name ILIKE '%Texas Youth Soccer%';
```

Record result: **Pre-config baseline: _______ games**

---

### Step 2 — Add TYSA to Config (< 5 min)

1. Open `crawl-gotsport-api-parallel.js` in the project root
2. Locate the USYS State Associations section (or create it with a comment header if it doesn't exist)
3. Add the TYSA entry:
   ```javascript
   // Texas Youth Soccer Association (USYS TX) — added 2026-04-[DD] by Presten
   [TYSA_ORG_ID], // TYSA — Texas statewide
   ```
   Replace `[TYSA_ORG_ID]` with the exact numeric value from the browser session.
4. Verify no trailing comma issues; entry should end with a comma if other entries follow in the array
5. Save the file

---

### Step 3 — Syntax Validation (< 2 min)

```bash
node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"
```

**Expected output:** `Syntax OK`  
**If error appears:** Stop. Document the error. Do NOT proceed. File SENTINEL queue item with the error message before retrying. Do not run the pipeline until syntax is confirmed clean.

---

### Step 4 — Duplicate Check (< 1 min)

```bash
grep -n "[TYSA_ORG_ID]" crawl-gotsport-api-parallel.js
```

Replace `[TYSA_ORG_ID]` with the actual numeric value.

**Expected:** Exactly 1 line returned.  
**If 2+ lines:** Duplicate entry — remove the redundant one. Re-run syntax check. Re-run duplicate check until exactly 1 line remains.

---

### Step 5 — Confirm Entry Written Correctly (< 1 min)

```bash
grep -A1 "Texas Youth Soccer" crawl-gotsport-api-parallel.js
```

**Expected:** Comment line + numeric entry matching the org-ID provided by Presten exactly.  
**If org-ID doesn't match:** Edit the file; repeat Steps 3–5.

---

### Step 6 — Sign May 1 Stage 1 Config Final Review (< 5 min)

Open: `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md`

In **Section 5: FORGE Certification**, update:

```
FORGE certifies that the Stage 1 org ID configuration listed in Section 1 has been reviewed
against all research documents filed through 2026-04-28. All Confirmed org IDs are verified active.
No contradictions were found. The configuration is deployment-ready for May 1.

FORGE certification date: 2026-04-[DATE]
TYSA org-ID confirmed: [VALUE]
Config syntax verified: YES
Signed: FORGE — [date and time of signing]
```

Also update Section 1 of the Config Review card: fill in the TYSA org-ID row (currently blank) with the confirmed org-ID and change Status from "Needs Browser Check" to "Confirmed."

---

### Step 7 — Note in Next Briefing

FORGE's next briefing must include:

```
- TYSA org-ID received: [value]
- Config add: COMPLETE (2026-04-[DD] at [HH:MM])
- Pre-config baseline: [N] existing TYSA-related games
- May 1 Stage 1 certification: SIGNED
- First-run monitoring: ready for May 1
- Expected first-run result: ≥ 5,000 new games from TYSA org-ID (GREEN floor)
```

---

## Section 3: First-Run Validation (Execute After May 1 Pipeline Runs)

Run after the first post-May-1 pipeline execution completes:

```sql
-- TYSA game ingestion check (primary)
SELECT 
  COUNT(*) AS total_games,
  MIN(game_date) AS earliest_game,
  MAX(game_date) AS latest_game,
  MAX(created_at) AS ingestion_timestamp
FROM games 
WHERE source_org_id = '[TYSA_ORG_ID]';
```

**GREEN:** total_games ≥ 5,000 — config add successful, org-ID is active  
**YELLOW:** total_games 500–4,999 — partial; TYSA org-ID may be scoped to state cup events only; investigate before Stage 2  
**RED:** total_games < 500 — flag immediately to SENTINEL; do not proceed with Stage 2 until root cause confirmed

```sql
-- Dedup check (run alongside primary)
SELECT source_id, COUNT(*) AS count
FROM games
WHERE source = 'gotsport'
  AND source_org_id = '[TYSA_ORG_ID]'
  AND created_at > NOW() - INTERVAL '24 hours'
GROUP BY source_id
HAVING COUNT(*) > 1;
```

Expected: 0 rows. Any rows = dedup failure — stop and notify SENTINEL.

Record results in `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` (Run 1, TYSA row) and in `Infrastructure/Pipeline Week 1 — Game Volume Actuals Report.md` (Section 2, May 1 TYSA row).

---

## Completion Record

Fill in after all steps are complete:

```
TYSA org-ID received:           ___________
Received from:                  Presten (browser session)
Date/time of receipt:           ___________
Date/time of config add:        ___________
Syntax check result:            PASSED / FAILED
Duplicate check result:         1 occurrence / OTHER (see notes)
Config Review Section 5 signed: YES / NO
Signed at:                      ___________

First-run validation (May 1):
  Total TYSA games ingested:    ___________
  Status:                       GREEN / YELLOW / RED
  First-run monitoring filed:   YES / NO
```

---

## References

- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config entry format and add procedure
- `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` — Section 5 to sign
- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — GREEN/YELLOW/RED thresholds for first run
- `Infrastructure/Pipeline Week 1 — Game Volume Actuals Report.md` — record first-run results here
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — companion monitoring log
- `Infrastructure/EDP + USL Academy — Config Activation Card.md` — parallel activation card (same browser session)
- `task-2026-04-26-stage2-authorization-trigger-document.md` — Stage 2 gates on May 1 Stage 1 certification

*FORGE — 2026-04-28*
