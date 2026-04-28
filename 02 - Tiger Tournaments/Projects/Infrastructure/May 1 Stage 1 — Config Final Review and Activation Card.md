---
title: May 1 Stage 1 — Config Final Review and Activation Card
tags: [infrastructure, gotsport, stage1, texas, config, pipeline, deployment, evo-draw]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: filed — CERTIFICATION BLOCKED on TYSA org-ID browser confirmation
task: task-2026-04-28-may1-stage1-config-final-review
due: 2026-04-30
---

# May 1 Stage 1 — Config Final Review and Activation Card

> [!warning] Certification Status: BLOCKED
> FORGE has completed all sections of this review. **FORGE cannot sign Section 5 certification until the TYSA browser session confirms the org-ID.** TYSA is the only Stage 1 entity. All other sections are complete and contradiction-free. Once Presten confirms the TYSA org_id, FORGE fills the activation command placeholder and signs Section 5 within 1 hour.
>
> **Presten action required:** Complete browser lookup at `system.gotsport.com` → search "Texas Youth Soccer" → extract org_id from URL parameter `?org_id=XXXX`. Share org_id with FORGE.

**Prepared by:** FORGE — 2026-04-28  
**FORGE certification:** PENDING — awaiting TYSA org_id  
**SENTINEL review:** PENDING FORGE certification  
**Deployment date:** May 1

---

## Section 1: Stage 1 Org-ID Manifest

Stage 1 is a single-entity pilot crawl. One org-ID. One state association. Intentionally narrow to validate pipeline behavior before broader expansion.

| Org ID | Club / Organization | State | League / Type | Status | Source Doc |
|--------|---------------------|-------|--------------|--------|------------|
| **[PENDING]** | Texas Youth Soccer Association (TYSA) | TX | USYS State Association | **Needs Browser Check** | `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` Section 2; `Infrastructure/GotSport Browser Lookup Session Prep Package.md` |

**Scope covered by TYSA org-ID (once confirmed):**
- All USYS-affiliated Texas leagues (Texas State League, Club Texas, TYSA State Cup, affiliated tournaments)
- Estimated 30,000–60,000 games/year at full GotSport penetration

**Stage 1 activation set (Confirmed org-IDs only):** 0 confirmed as of 2026-04-28  
**Stage 1 activation set (after browser session):** 1 (TYSA, once confirmed)

> [!info] FORGE definition: "Confirmed"
> An org-ID is Confirmed only when verified via browser lookup at `system.gotsport.com`. TYSA does not have a confirmed org-ID as of this filing. FORGE does not certify any org-ID that is still `pending-browser-lookup`. This is not a gap in FORGE's work — the browser session is the final unblocking action, and it requires Presten.

---

## Section 2: Config Contradiction Check

FORGE reviewed all org-ID research documents filed April 22–28. Findings:

- [x] No org-ID appears in both the active list and the inactive/flagged list — **PASS** (no org-IDs are confirmed yet; no contradictions possible)
- [x] No duplicate org-IDs in the Stage 1 set — **PASS** (Stage 1 is one entity)
- [x] TYSA org-ID status: **Needs Browser Check** — TYSA browser lookup is the first entry in `Infrastructure/GotSport Browser Lookup Session Prep Package.md`. FORGE has pre-staged the config entry in `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` with `[TBD-ORG-ID]` placeholder.
- [x] No org-IDs flagged as "wrong state" or "wrong club" in research documents — **PASS** (TYSA = Texas state association; correct entity for Stage 1 TX pilot)
- [x] VA / CO / AZ org-IDs: **Not in Stage 1** — These are Stage 3 entities, pending browser confirmation. Correctly excluded from Stage 1. Source: `Infrastructure/GotSport Org-ID Research — VA, CO, AZ.md`
- [x] Pacific Northwest org-IDs: **Not in Stage 1** — Stage 3 entities. Correctly excluded. Source: `Infrastructure/Pacific Northwest GotSport Org-ID Research.md`

**Contradiction log:** None. No contradictions found across all research documents. Stage 1 is a single-entity config and the entity is correctly identified.

**Known gap (not a contradiction):** TYSA org_id is unconfirmed. This is a data gap (browser session not yet run), not a contradiction. Resolution path is clear: browser session → FORGE fills placeholder → FORGE signs Section 5.

---

## Section 3: Expected Game Volume

| State | Org-IDs Activating | Expected New Games (First 7 Days) | Expected New Games (Annual) | Calibration League? |
|-------|-------------------|----------------------------------|----------------------------|---------------------|
| TX | 1 (TYSA — pending confirmation) | 3,500–7,000 (est. 60% of annual games played by May) | 30,000–60,000 | NO — state association (multiple tiers); some ECNL-affiliated TX teams may appear |
| **Total Stage 1** | **1** | **3,500–7,000** | **30,000–60,000** | — |

**Volume source:** `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` Section 2  
**GREEN floor for first run:** ≥ 5,000 new games ingested (per Expected Output Spec Section 3)  
**YELLOW band:** 500–4,999 new games — investigate before authorizing Stage 2  
**RED:** < 500 new games — diagnose before proceeding  

**ELO note:** These volume figures are passed to ELO's May 1 Rankings Monitoring Plan. ELO uses the GREEN floor (5,000) as the minimum-expected ingestion threshold when assessing ranking impact on May 1.

---

## Section 4: Activation Command

> [!warning] TYSA org-ID placeholder — fill before May 1
> Replace `[TYSA-ORG-ID]` with the confirmed numeric org_id from the browser session.

**Step 1 — Pre-activation: confirm baseline game count**
```sql
-- Run BEFORE activation to establish baseline
SELECT
  COUNT(*) AS total_games,
  MAX(ingested_at) AS most_recent_ingestion
FROM games
WHERE source = 'gotsport';
```
Record result and timestamp.

**Step 2 — Config add: open `crawl-gotsport-api-parallel.js` and add TYSA entry**

Using the pre-written entry from `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` → Group A: TYSA. Replace `[TBD-ORG-ID]` with confirmed org_id. Paste under the appropriate config section.

```javascript
// Texas Youth Soccer Association (TYSA) — added [DATE] by Presten
[TYSA-ORG-ID], // TYSA — Texas statewide (USYS state association)
```

**Step 3 — Syntax verification (run before any pipeline execution)**
```bash
node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"
```
Expected output: `Syntax OK`

**Step 4 — Execute Stage 1 pipeline run**
```bash
node crawl-gotsport-api-parallel.js
```
Or trigger via existing daily pipeline mechanism. Check logs for any 4xx/5xx errors from GotSport API.

**Step 5 — Post-activation check (run within 15 minutes of first pipeline run completing)**
```sql
-- Confirm Stage 1 games ingested
SELECT source_org_id, COUNT(*) AS games_ingested
FROM games
WHERE source = 'gotsport'
  AND ingested_at >= NOW() - INTERVAL '24 hours'
GROUP BY source_org_id
ORDER BY games_ingested DESC;
```
Expected: One row for TYSA's org_id with count ≥ 5,000 (GREEN) or ≥ 500 (YELLOW).

**Step 6 — Dedup verification**
```sql
-- Confirm no duplicate games from Stage 1 run
SELECT source_id, COUNT(*) AS count
FROM games
WHERE source = 'gotsport'
  AND ingested_at >= NOW() - INTERVAL '24 hours'
GROUP BY source_id
HAVING COUNT(*) > 1;
```
Expected: 0 rows.

---

## Section 5: FORGE Certification

> [!warning] PENDING — Cannot certify until TYSA org-ID confirmed
> FORGE certifies only Confirmed org-IDs (see Section 1 definition). TYSA's org-ID is `pending-browser-lookup` as of 2026-04-28. Once Presten confirms the org-ID, FORGE replaces this warning block with the certification statement below and signs.

**Certification statement (to be signed when TYSA org_id confirmed):**

> FORGE certifies that the Stage 1 org-ID configuration listed in Section 1 has been reviewed against all research documents filed through 2026-04-28. The TYSA org-ID has been confirmed via browser lookup. No contradictions were found across all Stage 1 research documents. The configuration is deployment-ready for May 1.

FORGE certification date: _[fill when TYSA org_id confirmed]_  
FORGE signature: _FORGE_

SENTINEL review: **PENDING FORGE CERTIFICATION**  
SENTINEL signature: _________________  
SENTINEL review date: _________________

---

## Section 6: Known Gaps and Post-May-1 Actions

| Gap | Reason Not in Stage 1 | Next Action |
|-----|----------------------|-------------|
| **TYSA org-ID unconfirmed** | Browser session not yet executed | Presten completes browser session → shares org_id → FORGE fills Section 4 placeholder, updates Org-ID Master Reference, signs Section 5 certification within 1 hour |
| All Stage 2 entities (USL Academy, EDP, Elite 64, Cal North, Cal South, FYSA, ENYYSA, NJYSA, WSYSA, EPYSA) | Pending browser session + SENTINEL Stage 2 authorization | After Stage 1 GREEN result + SENTINEL authorization gate passes → FORGE adds Stage 2 org-IDs within 1 session |
| Stage 3 entities (VA, CO, AZ) | Pending browser session | After Stage 2 confirmed stable → Stage 3 authorization in separate SENTINEL gate |
| NAL | Platform unconfirmed | Run 15-minute GotSport search + website check. If GotSport confirmed: reassess as Stage 2 config add. If non-GotSport: defer post-DSS. |

---

## FORGE Post-Browser-Session Action Sequence

When Presten shares TYSA org-ID:

1. Open `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md`
2. Locate TYSA entry → replace `[TBD-ORG-ID]` with confirmed org_id
3. In THIS document: update Section 1 manifest (Org ID column, Status → `Confirmed`)
4. In THIS document: update Section 4 activation command (replace `[TYSA-ORG-ID]`)
5. In THIS document: replace Section 5 warning with certification statement, fill date, sign
6. Update `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` → move TYSA from Section 2 to Section 1, update Section 4 Status Summary
7. Notify SENTINEL that Section 5 certification is ready for SENTINEL signature

Target: Steps 1–7 complete within 1 hour of org-ID receipt.

---

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — TYSA status (Section 2 → moves to Section 1 after confirmation)
- `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` — pre-written TYSA config entry
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — TYSA lookup instructions
- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — volume estimates and GREEN/YELLOW/RED criteria
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — full May 1 execution guide
- `Infrastructure/May 1 First-Run Validation Checklist.md` — post-activation validation checklist
- `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md` — Stage 2 gate unlocked after Stage 1 GREEN

*FORGE — 2026-04-28*
