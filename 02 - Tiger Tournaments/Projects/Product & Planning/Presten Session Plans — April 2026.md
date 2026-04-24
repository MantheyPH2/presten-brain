---
title: Presten Session Plans — April 2026
tags: [planning, execution, presten, sprint, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: ready
task: task-2026-04-24-presten-session-plans
---

# Presten Session Plans — April 2026

> [!tip] Decision Guide — Read This First
> 
> ```
> If you have 15 minutes: Run the USA Rank SQL audit (30-min standalone block below).
> If you have 1 hour:     Run the 1-Hour Block below.
> If you have 3 hours:    Run the 3-Hour Block below.
> If you have a full day: 3-Hour Block in the morning + Event Strength session in the afternoon.
> If you are unsure:      Start at Step 1 of the 1-Hour Block.
> ```

> [!warning] Source Path Alert
> The GA ASPIRE tier fix referenced as a 1-Hour Block step requires `Reports/ga-aspire-tier-fix-2026-04-[date].md`. **This file does not exist in the vault as of 2026-04-24.** Steps that depend on it are marked ⚠️ BLOCKED. All other steps are executable. SENTINEL should confirm whether ELO has written this document or whether the fix is embedded elsewhere (e.g., in `Rankings/calibration-fix-pre-deploy-2026-04-23.md`).

---

## 30-Minute Standalone: USA Rank SQL Audit

**Total time: 30–45 minutes. Unblocks: ELO calibration approval, Football Academy NJ merge question.**

This can be run at any time, independently of other blocks. No code change required.

### What to do

Open a psql session: `psql -U p -d youth_soccer -h localhost -p 5432`

All 16 queries are pre-written in:
`02 - Tiger Tournaments/Projects/Rankings/USA Rank Ranking Accuracy Audit — April 2026.md`

Run each fuzzy lookup (e.g., `ILIKE '%Football Academy%'`), then fill in the Evo Draw rank and rating columns in:
`02 - Tiger Tournaments/Projects/Rankings/USA Rank Comparison 2026-04-23.md`

Pay particular attention to the Football Academy NJ / FA 12G Black identity query — are they one team or two in our DB? The exact SQL is in the audit document.

**Expected output:** A completed comparison table showing Evo Draw rank vs USA Rank for 16 reference teams. Any team not found: note "NOT FOUND" + check if games exist in the `games` table.

**How to verify it worked:** The `USA Rank Comparison 2026-04-23.md` document has no `???` entries in the Evo Draw column.

**Time estimate:** 30–45 minutes. Pure data retrieval — no code changes.

---

## Session Block 1: 1-Hour Block — Quick Wins

**Total time: ~60 minutes. Unblocks: June 1 risk (partially), calibration approval, backoff deployment.**

> [!note] Current Status
> - Step 1 (ECNL P0): Deferred — script not written until May 1. Action today is review-only (5 min). See note below.
> - Step 2 (GA ASPIRE): ⚠️ BLOCKED — source document not found. Skip until SENTINEL provides the path.
> - Step 3 (backoff): This is the most actionable step in this block. Do this one.

### Step 1: Confirm ECNL Dedup P0 Fix Spec — 5 minutes

**Status:** No code change needed today. `ecnl-transition-dedup.js` does not exist yet — FORGE writes it on May 1.

Read `02 - Tiger Tournaments/Projects/Reports/ecnl-dedup-verified-fix-2026-04-24.md` to confirm you understand the required INSERT pattern. The P0 is documented and proactively closed — when FORGE writes the script in May, it will include `verified = true` in all team_merges INSERTs.

**Calendar note:** Block 15–30 minutes on or around May 5–10 to review the new script and confirm the INSERT pattern before the June 1 run window.

### Step 2: Apply GA ASPIRE Tier Fix — ⚠️ BLOCKED

⚠️ **Source document not found.** The file `Reports/ga-aspire-tier-fix-2026-04-[date].md` does not exist in the vault. Skip this step until SENTINEL confirms the document path.

Possible alternative: the relevant fix may be in `Rankings/calibration-fix-pre-deploy-2026-04-23.md` (ELO's calibration pre-deploy checklist). Check Section 1 of that document for GA-related tier changes.

### Step 3: Apply Exponential Backoff Fix to Sync Runner — ~60 minutes

**File to modify:** `crawl-gotsport-api-parallel.js`  
**Full spec:** `02 - Tiger Tournaments/Projects/Reports/backoff-implementation-2026-04-24.md` — copy-paste ready code is in this document.

**What to do:**

1. Open `crawl-gotsport-api-parallel.js`
2. Replace the `INTER_REQUEST_DELAY_MS` constant:
   ```javascript
   // BEFORE
   const INTER_REQUEST_DELAY_MS = 1200;
   
   // AFTER
   const INTER_REQUEST_DELAY_MS = 600;
   const BACKOFF_BASE_MS = 1000;
   const BACKOFF_CAP_MS = 32000;
   const BACKOFF_JITTER_PCT = 0.20;
   const MAX_RETRIES_PER_TEAM = 5;
   ```
3. Replace `fetchTeamMatches()` with `fetchWithBackoff()` — full code is in `Reports/backoff-implementation-2026-04-24.md`
4. Add the backoff counter variables and `BACKOFF_DAILY_SUMMARY` log line (also in the report)

**How to verify it worked:** Run the sync runner against a single test team ID:
```bash
GSAPI_TEAM_IDS="<any-valid-id>" node crawl-gotsport-api-parallel.js
```
Confirm the run produces a `SYNC_RUN_SUMMARY` log line. If a 429 occurs during the test, confirm `API_RETRY` log events appear.

**Time box:** 60 minutes. If you hit an unexpected issue, stop and note the blocker — do not spend more than 60 minutes.

**Exit check:** After completing Step 3, confirm the Stage 1 backfill pilot can start immediately (10 teams, 15 minutes). The runbook is at `Infrastructure/Stale Team Backfill Runbook — April 2026.md`.

---

## Session Block 2: 3-Hour Block — Critical Path Unblock

**Total time: ~3 hours. Includes everything actionable from the 1-Hour Block, plus TX org-ID lookup and Stage 1 backfill.**

Complete Step 3 from the 1-Hour Block first, then:

### Step 4: TX Org-ID Browser Lookup + Stage 1 Backfill Pilot — ~90 minutes

**Source:** `02 - Tiger Tournaments/Projects/Data Sources/GotSport USYS Org-ID Master List.md` — lookup procedure for each state is documented with the method.

**Part A: TX Org-ID Lookup (~15 minutes)**

In a browser, navigate to the GotSport Texas Youth Soccer Association org page. Look for the org ID in the URL (e.g., `/events/org/XXXX`) or in event listings. Common path: `gotsport.com/organizations` or search for "Texas Youth Soccer Association."

Update the JSON config block in `GotSport USYS Org-ID Master List.md`:
```json
{ "state": "TX", "org_name": "Texas Youth Soccer Association", "org_id": "[ID]", "priority": 1 }
```

While in the browser, also collect FL, NY, and NJ org IDs — they're P1 states and the lookups are identical.

**Part B: Stage 1 Backfill Pilot — 10 Teams (~20 minutes including run)**

Pre-conditions before starting (all should be true after Step 3):
- [ ] `fetchWithBackoff()` confirmed in `crawl-gotsport-api-parallel.js`
- [ ] `Logs/dlq/` directory exists
- [ ] No other crawl job running (`ps aux | grep crawl`)

Run the 3 pre-backfill verification queries from `Infrastructure/Stale Team Backfill Runbook — April 2026.md` (Step 1, Queries A/B/C). Record the baseline counts.

Then run the Stage 1 pilot:
```bash
head -11 Reports/stale-teams-2026-04-21.csv | tail -10 | cut -d',' -f1 > /tmp/pilot-ids.txt
GSAPI_TEAM_IDS_FILE=/tmp/pilot-ids.txt node crawl-gotsport-api-parallel.js
```

Monitor logs for 15 minutes. Run post-Stage 1 verification queries (Stage 1 section of the runbook).

Go/no-go for Stage 2 is in the runbook — if Stage 1 passes, Stage 2 (50 teams) can run same day or next day.

**IMPORTANT:** Do NOT run Stage 3 (480 teams) on the same night as the first daily pipeline run. Sequencing: backoff deploy → Stage 1 → Stage 2 → daily pipeline first run → Stage 3 overnight.

**Time box:** 90 minutes total for Part A + Part B.

### Step 5: Calibration Fix Review (Optional, if time allows) — 30 minutes

Review `Rankings/calibration-fix-pre-deploy-2026-04-23.md`. The calibration changes for U12/U13/U14/U19 are specced and ready but require your approval before deployment. If you're ready to approve, sign off in `Queue/pending-2026-04-22-calibration-approval.md`.

**Do NOT deploy before DSS (May 16+).** This is a review-only step.

---

## Session Block 3: 6-Hour Block — DSS Feature Work

**Total time: 6–8 hours. Includes the 3-Hour Block plus a full Event Strength implementation session.**

Complete the 3-Hour Block first (Steps 3 and 4). Then:

### Step 6: Event Strength — Full Implementation Session

**Source:** `02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings — Implementation Plan.md`

This is a self-contained 7-hour session. Do not compress it.

**Pre-flight before starting:**
1. Read `02 - Tiger Tournaments/Projects/Rankings/Event Strength Post-Compute Validation — 2026-04-24.md` — understand what correct output looks like before implementing.
2. Confirm `compute-rankings.js` is running cleanly and the last ranking computation succeeded.

If you are starting this after completing the 3-Hour Block in the same day, take a break first.

---

## Verified Source Paths

| Document | Vault Path | Exists? |
|---|---|---|
| Backoff implementation spec | `Reports/backoff-implementation-2026-04-24.md` | ✅ |
| ECNL dedup P0 fix note | `Reports/ecnl-dedup-verified-fix-2026-04-24.md` | ✅ |
| GA ASPIRE tier fix SQL | `Reports/ga-aspire-tier-fix-2026-04-[date].md` | ❌ NOT FOUND |
| USA Rank accuracy audit | `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` | ✅ |
| USA Rank comparison table | `Rankings/USA Rank Comparison 2026-04-23.md` | ✅ |
| GotSport Org-ID Master List | `Data Sources/GotSport USYS Org-ID Master List.md` | ✅ |
| Stale team backfill runbook | `Infrastructure/Stale Team Backfill Runbook — April 2026.md` | ✅ |
| Event Strength implementation plan | `Rankings/Event Strength Ratings — Implementation Plan.md` | ✅ |
| Event Strength post-compute validation | `Rankings/Event Strength Post-Compute Validation — 2026-04-24.md` | ✅ |
| Calibration pre-deploy checklist | `Rankings/calibration-fix-pre-deploy-2026-04-23.md` | ✅ |

**One unresolved reference:** GA ASPIRE tier fix document. SENTINEL should provide the correct path before Presten attempts the 1-Hour Block Step 2. All other paths confirmed present.

---

## Definition of Done

This document is complete when:
- [ ] Backoff fix deployed (`crawl-gotsport-api-parallel.js` updated) — unblocks backfill
- [ ] Stage 1 pilot passed — confirms backoff works in production
- [ ] TX org ID confirmed in the Org-ID Master List — unblocks org-level discovery
- [ ] USA Rank comparison table filled in — unblocks calibration approval
- [ ] GA ASPIRE source document located (SENTINEL action) — unblocks that fix

---

*FORGE — 2026-04-24*
