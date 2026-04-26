---
title: April 29 FORGE Session Checklist
tags: [infrastructure, pipeline, session-checklist, forge, evo-draw]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — use on April 29
---

# April 29 FORGE Session Checklist

```
Session Date: April 29, 2026
Prerequisite: April 28 execution log confirmed filed (check Infrastructure/April 28 Execution Log.md before starting)
Total estimated time: ~45 minutes for Steps 1–4. Steps 5–6 are <10 minutes each.
```

---

## Step 1: Confirm April 28 Execution Log (5 min)

**Prerequisite:** April 28 session ran.

- [ ] Open `Infrastructure/April 28 Execution Log.md`
- [ ] Verify all 3 Step ✅ boxes are filled (GA ASPIRE UPDATE, ecnl_verified ALTER TABLE, rankings recompute)
- [ ] Read SENTINEL Disposition section — is Event Strength Phase 1 authorized or held?

**If HELD:** Stop here. File a Queue item at `10 - Agents/SENTINEL/Queue/` flagging the hold and do not proceed to Step 2 until SENTINEL resolves.

**Prerequisite for Step 2:** All 3 steps confirmed ✅.

---

## Step 2: Source Baseline Section 4 — Run 3 Queries (15 min)

**Prerequisite:** Step 1 confirmed ✅.

**Reference:** `task-2026-04-26-post-april28-source-baseline-section4-update`
**Target file:** `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` → Section 4

Run in this order in psql:

### Query A — GA ASPIRE Event Tier Verification

```sql
SELECT event_tier, COUNT(*) as game_count
FROM games
WHERE event_name ILIKE '%ga aspire%' OR event_tier = 'ga_aspire'
GROUP BY event_tier
ORDER BY game_count DESC;
```

**PASS:** All rows show `event_tier = 'ga_aspire'`. Zero rows where `event_tier = 'ga'` and event name indicates GA ASPIRE.
**FAIL:** Any row shows `event_tier = 'ga'` for ASPIRE events → flag to SENTINEL immediately before continuing.

### Query B — ecnl_verified Column Confirmation

```sql
SELECT column_name, data_type, column_default
FROM information_schema.columns
WHERE table_name = 'games' AND column_name = 'ecnl_verified';
```

**PASS:** One row returned; `data_type = 'boolean'`; `column_default = 'false'`.
**FAIL:** No row returned (column not added) → flag to SENTINEL. Do not proceed with Step 3.

### Query C — Post-Recompute Game Count Delta

```sql
SELECT COUNT(*) as total_games, MAX(updated_at) as last_updated
FROM games;
```

Record: total games now, compare to pre-April-28 baseline from Section 2 of the Baseline document. A delta of 0 on total count is expected (only event_tier and ecnl_verified values changed, no rows added or removed).

**Unexpected large delta (positive or negative):** Flag in baseline document notes and file a briefing flag. Do not escalate to SENTINEL unless delta is >1% of total game count or negative.

**After all three queries:** File results in `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` Section 4. Mark `task-2026-04-26-post-april28-source-baseline-section4-update` as completed.

---

## Step 3: FM1/FM2 Audit Document Updates (10 min if results in; skip if not)

- [ ] Open `Infrastructure/Pipeline Code Review Results.md`
- [ ] Check if FM1 Path Letter and FM2 Path Letter are filled in by Presten

**If path letters ARE filed:**
1. Open `Infrastructure/FM1-FM2 Contingency Action Plan.md`
2. Locate the matching row in the Combined Outcome Matrix (Section 3) for the FM1 × FM2 combination
3. Execute the FORGE action for that row
4. Update both audit documents per the Outcome Update Protocol (`task-2026-04-26-audit-documents-outcome-update-protocol.md`)
5. Mark `task-2026-04-26-audit-documents-outcome-update-protocol.md` as completed

**If path letters are NOT filed:**
File a Queue item at `10 - Agents/FORGE/Queue/pending-2026-04-29-fm1-fm2-still-unresolved.md`:
```
FM1/FM2 results not received. Pipeline Code Review Results.md is still blank as of April 29.
May 1 deployment confidence gate is not met.
SENTINEL must escalate to Presten before April 28 DB session / May 1 deployment decision.
Gate at risk: May 1 Pre-Authorization Checklist Section 3 (run-daily.sh) and Section 4 (Stage 1 status).
```

---

## Step 4: First-Week Monitoring Log — Pre-Population (10 min)

**Reference:** `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md`

Pre-fill what can be completed before May 1 runs:

**Section 1, Row 1 (May 1):**
- Date: 2026-05-01
- Run Status: *(leave blank — fill after May 1 run)*
- New Games Ingested: *(leave blank)*
- Total Games in DB: *(leave blank — but note today's Section 2 count from Step 2 Query C as the pre-run baseline)*
- GREEN/YELLOW/RED: *(leave blank)*
- Notes: "First production run. GREEN/YELLOW/RED to be filled post-run. Pre-run total games baseline: [paste count from Step 2 Query C]."

**Section 4 (Issues Log):**
Add one entry:
```
Date: 2026-04-29
Issue: Pre-population entry — monitoring begins May 1. No issues to report pre-run.
Severity: N/A
Resolution: N/A
Resolved: N/A
```

---

## Step 5: ELO Coordination — CP1 Staleness Check (5 min)

- [ ] Open `10 - Agents/ELO/Briefings/` — find the April 29 ELO briefing
- [ ] Confirm ELO filed CP1 result from the ECNL RL staleness check

**If CP1 = PASS (ECNL RL ≤ 30 days since most recent game):** No FORGE action required. Note in FORGE briefing.

**If CP1 = FAIL (ECNL RL > 30 days):** File a FORGE Queue item at `10 - Agents/FORGE/Queue/pending-2026-04-29-ecnl-rl-staleness-fail.md`:
```
CP1 FAIL: ECNL RL staleness check failed — last game > 30 days ago.
Implication: ECNL Option 1 (rating continuity via RL overlap) is disqualified.
ECNL migration must escalate to ELO Option 2 (re-tag historical games) or Option 3 (clean break).
FORGE will make no ECNL RL config changes until ELO files Option 2/3 selection and SENTINEL authorizes.
```

**If ELO April 29 briefing is not yet filed:** Note in FORGE briefing: "CP1 result not received from ELO. Will check at next FORGE session."

---

## Checklist Summary

| Step | Time | Prerequisite | Output |
|------|------|-------------|--------|
| 1: Confirm Execution Log | 5 min | April 28 session ran | Gate check — HELD = stop |
| 2: Source Baseline Section 4 | 15 min | Step 1 ✅ | Source Ingestion Baseline — Section 4 filled |
| 3: FM1/FM2 Audit Updates | 10 min | Path letters in Code Review Results | Audit docs updated OR Queue escalation filed |
| 4: Pre-populate Monitoring Log | 10 min | Step 2 Query C result | First-Week Monitoring Log Row 1 pre-filled |
| 5: ELO CP1 coordination | 5 min | ELO April 29 briefing filed | Queue item if CP1 fails; note if ELO not yet filed |

**Total:** ~45 minutes.

---

## References

- `Infrastructure/April 28 Execution Log.md` — Step 1 gate
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — Step 2 target
- `Infrastructure/Pipeline Code Review Results.md` — Step 3 source
- `Infrastructure/FM1-FM2 Contingency Action Plan.md` — Step 3 routing
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — Step 4 target
- `task-2026-04-26-post-april28-source-baseline-section4-update` — Step 2 task file
- `task-2026-04-26-audit-documents-outcome-update-protocol` — Step 3 task file

*FORGE — 2026-04-26*
