---
title: EDP + USL Academy — Config Activation Card
tags: [infrastructure, gotsport, edp, usl-academy, config, pipeline, stage2, forge]
created: 2026-04-28
author: FORGE
status: ready — awaiting org-ID delivery from browser session
---

# EDP + USL Academy — Config Activation Card

> **Purpose:** Live execution reference for the moment Presten delivers EDP and USL Academy org-IDs. No design decisions at execution time. Fill the two placeholders, run validation, done. Estimated execution time after org-ID receipt: < 1 hour.

---

## 1. Pre-Activation Gate (5 minutes — run before touching config)

Both checks must pass. If either fails: stop, do not edit config, notify SENTINEL via queue (`category: alert, priority: urgent`).

| Check | How to Verify | Pass Condition |
|-------|--------------|----------------|
| Stage 1 TX crawl complete and verified | Read `Infrastructure/Stage 1 — FORGE Verification Statement.md` — confirm "Overall Stage 1 result: PASS" | Verification statement filed and result = PASS |
| No active pipeline run in progress | Check process list: `ps aux \| grep crawl` from project root | No active crawl PID |

**If Stage 1 has not yet run:** EDP and USL Academy can still be added to config — they will be ingested on the first pipeline run. The gate is informational. Only delay if Stage 1 is actively running (avoid config edits mid-crawl).

---

## 2. EDP Config Entry

**Config file:** `crawl-gotsport-api-parallel.js` (project root — same directory as `run-daily.sh` and `run-overnight.sh`)

**Step 1:** Confirm filename before editing:
```bash
ls *.js | grep crawl-gotsport
```
Expected: `crawl-gotsport-api-parallel.js`

**Step 2:** Open config file. Do not use a GUI editor that may introduce encoding changes.

**Step 3:** Locate the `// EDP Soccer` section, or create it if absent. Add this entry, replacing `[EDP_ORG_ID]` with the confirmed numeric org-ID from the browser session:

```javascript
// EDP Soccer — added 2026-04-28 by Presten
[EDP_ORG_ID], // EDP Soccer — national
```

**Step 4 — Syntax check (run immediately after adding EDP entry):**
```bash
node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"
```
Expected: `Syntax OK`. If a syntax error appears: fix before proceeding. A syntax error in this file causes all crawl runs to fail.

---

## 3. USL Academy Config Entry

**Same config file:** `crawl-gotsport-api-parallel.js`

**Step 1:** Locate the `// USL Academy` section, or create it if absent.

**Step 2:** Add this entry, replacing `[USL_ACADEMY_ORG_ID]` with the confirmed numeric org-ID:

```javascript
// USL Academy — added 2026-04-28 by Presten
[USL_ACADEMY_ORG_ID], // USL Academy — national
```

If the browser session returned multiple USL Academy org-IDs (possible — USL Academy may have regional GotSport entities), add each as a separate line in the same section:

```javascript
// USL Academy — added 2026-04-28 by Presten
[USL_ACADEMY_ORG_ID_1], // USL Academy — East
[USL_ACADEMY_ORG_ID_2], // USL Academy — West
```

**Step 3 — Syntax check after both EDP and USL Academy entries are added:**
```bash
node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"
```

---

## 4. Post-Add Validation SQL

Run all three queries immediately after config edits, before the next crawl runs. These verify the database state, not game ingestion (new games appear 24–48 hours after the next scheduled crawl).

**Q1 — EDP: Confirm org-ID is not already ingesting via another path**
```sql
SELECT source_org_id, COUNT(*) AS existing_games
FROM games
WHERE source_org_id = '[EDP_ORG_ID]'
GROUP BY source_org_id;
```
**Expected result:** 0 rows.
**If rows exist:** EDP org-ID is already ingesting via a different path. Do not create a duplicate crawl entry. Investigate existing ingestion and notify SENTINEL before proceeding.

---

**Q2 — USL Academy: Confirm org-ID is not already ingesting via another path**
```sql
SELECT source_org_id, COUNT(*) AS existing_games
FROM games
WHERE source_org_id = '[USL_ACADEMY_ORG_ID]'
GROUP BY source_org_id;
```
**Expected result:** 0 rows.
**If rows exist:** Same action as Q1.

---

**Q3 — Config duplicate check (confirm each org-ID appears exactly once in file)**
```bash
grep -c "[EDP_ORG_ID]" crawl-gotsport-api-parallel.js
grep -c "[USL_ACADEMY_ORG_ID]" crawl-gotsport-api-parallel.js
```
**Expected result:** `1` for each command.
**If > 1:** Remove the duplicate entry before the next crawl runs. A double entry will cause double-crawling and inflated game counts.

---

**All three pass = activation confirmed.** Log results using Section 5 template.

---

## 5. Briefing Entry Template

Paste into the next FORGE briefing after activation. Replace all bracketed placeholders:

> EDP org-ID `[EDP_ORG_ID]` added to `crawl-gotsport-api-parallel.js` on [DATE_EXECUTED]; post-add validation (Q1–Q3) passed; first crawl expected at next scheduled pipeline run. USL Academy org-ID `[USL_ACADEMY_ORG_ID]` added in the same session; post-add validation passed; first crawl expected at next scheduled pipeline run. Source Gap Inventory updated for both sources.

After filing: update `Infrastructure/Source Gap Inventory — April 2026.md` — set EDP and USL Academy rows to status `Pending first crawl`, org-ID confirmed.

---

## 6. Stage 2 Trigger Condition

If Stage 1 TX crawl has verified ≥ 5,000 games **and** SENTINEL has issued written Stage 2 authorization, FORGE may proceed to Stage 2 org-ID expansion immediately after EDP + USL Academy activation. Reference: `Infrastructure/Stage 1 to Stage 2 — Transition Protocol.md`.

EDP and USL Academy activation is a Stage 2 action. SENTINEL authorization is required before config edits are made. Confirm authorization is in writing in the Stage 2 Authorization Trigger Document before proceeding.

---

## SENTINEL Verification

SENTINEL can verify EDP + USL Academy activation from this card alone:
1. Config edits confirmed in file (Q3 returns `1` for each org-ID)
2. No pre-existing ingestion (Q1 and Q2 return 0 rows)
3. Briefing entry filed referencing this card
4. No syntax errors in config file post-edit

---

## References

- `Infrastructure/Stage 1 Post-Run Verification Protocol.md` — pre-activation gate (Section 1 of this card)
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — full config structure, syntax, and entry templates
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — `games` table schema reference for Q1/Q2
- `Infrastructure/Non-GotSport Source Implementation Priority List.md` — confirms EDP and USL Academy are ranked 1 and 2
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — SENTINEL written authorization required before activation
- `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` — Stage 2 gate conditions

*FORGE — 2026-04-28*
