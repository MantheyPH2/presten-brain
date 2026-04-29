---
title: FM2 Audit — ecnl_verified TGS Scraper Write Path (Pre-Draft)
tags: [infrastructure, pipeline, ecnl, ecnl-verified, audit, fm2, forge]
created: 2026-04-29
updated: 2026-04-29
author: FORGE
status: pre-draft — awaiting Pipeline Code Review Results from Presten
task: task-2026-04-28-fm1-fm2-audit-pre-draft
---

# FM2 Audit — ecnl_verified TGS Scraper Write Path

> **STATUS: PRE-DRAFT — Awaiting Pipeline Code Review Results from Presten**
> Fill [FILL: ...] placeholders, remove this header, and move to final path `Infrastructure/FM2 Audit.md` on receipt.
>
> **Time to complete on receipt:** < 30 minutes (static sections are pre-populated; only result cells and verdict require Presten's data).

---

## Section 1: What FM2 Does

**FM2 = ecnl_verified field assignment in the TGS scraper** — the logic that sets `ecnl_verified = TRUE` on new ECNL game records at write time.

| Field | Value |
|-------|-------|
| Scope | TGS scraper game INSERT/upsert logic |
| Expected location | TGS scraper module (likely `tgs-scraper.js` or `crawlers/tgs/index.js`) |
| Trigger | Executes when a TGS or TGS AthleteOne game record is assembled and written to the `games` table |
| Field assigned | `ecnl_verified BOOLEAN` on the `games` table |
| Correct behavior | Games with `source IN ('tgs', 'tgs_athleteone')` AND `event_tier = 'ecnl'` must receive `ecnl_verified = TRUE` at write time |

**Why this matters:** The April 28 Step 2b backfill (`UPDATE games SET ecnl_verified = TRUE WHERE source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'`) fixes all historical records. If the scraper does not write `ecnl_verified = TRUE` at ingest time for new games, every ECNL game ingested after April 28 defaults to `ecnl_verified = FALSE`. The column becomes permanently inaccurate for all future data, degrading ECNL migration readiness and ELO's CP1 calibration.

**Reference audit document:** `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md`

---

## Section 2: What the Code Review Is Checking

FM2 checks (from `Infrastructure/Pipeline Code Review Results.md`):

| Check ID | What Presten Runs | What Correct Behavior Looks Like |
|----------|-------------------|----------------------------------|
| FM2-1 | `grep -rn "ecnl_verified" .` (codebase root) | Returns at least one file:line in the TGS scraper's INSERT/upsert block — not just a migration file or comment |
| FM2-2 | Locate the assignment; confirm condition matches backfill | Condition is `source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'` — not broader (all TGS) or narrower |
| FM2-3 | Confirm `ecnl_verified` is included in any `ON CONFLICT DO UPDATE SET` clause | If present and set to the correct expression: scraper is consistent; if absent from ON CONFLICT: re-scraping won't correct historical records |

**Assignment that must appear in the INSERT payload:**
```javascript
// In TGS scraper game write logic — within INSERT payload object
ecnl_verified: (
  ['tgs', 'tgs_athleteone'].includes(source) &&
  event_tier === 'ecnl'
),
```

---

## Section 3: Result Table

| Check | Presten's Finding | File:Line | Status |
|-------|-------------------|-----------|--------|
| FM2-1: `ecnl_verified` string found in TGS scraper | [FILL: from Pipeline Code Review Results] | [FILL: file:line] | [FILL: FOUND / NOT FOUND] |
| FM2-2: Assignment condition matches backfill criteria | [FILL: from Pipeline Code Review Results] | [FILL: file:line or N/A] | [FILL: MATCHES / MISMATCH / NOT FOUND] |
| FM2-3: ON CONFLICT SET includes ecnl_verified | [FILL: from Pipeline Code Review Results] | [FILL: file:line or N/A] | [FILL: YES / NO / N/A] |
| FM2 Path determined | [FILL: A / B / C] | — | — |

**FM2 Path definitions:**
- **Path A:** `ecnl_verified` is already set correctly in the TGS scraper at write time → no code change needed
- **Path B:** `ecnl_verified` is absent from the scraper's INSERT payload → simple field addition required (≤ 5 lines)
- **Path C:** Scraper architecture prevents simple field addition → structural change required, SENTINEL escalation; Stage 2 authorization held

---

## Section 4: Verdict

**FM2 Path: [FILL: A / B / C]**

| Verdict Criterion | Result |
|-------------------|--------|
| New ECNL games receive `ecnl_verified = TRUE` at ingest | [FILL: YES / NO / UNKNOWN] |
| Code change required before May 1 | [FILL: YES (Path B/C) / NO (Path A)] |
| Backfill condition consistency confirmed | [FILL: YES / NO / UNKNOWN] |
| Audit status after this filing | [FILL: CLOSED (Path A) / CONDITIONAL CLOSED (Path B fix confirmed) / ESCALATED (Path C)] |

**Pass criteria:** FM2 = Path A, OR FM2 = Path B with fix confirmed committed and deployed before May 1.
**Fail criteria:** FM2 = Path C, OR FM2 = Path B fix NOT deployed before May 1 pipeline launch.

**ELO dependency:** If FM2 = Path B or C, notify ELO immediately. ECNL CP1 staleness check and ECNL migration readiness (Option 2) both depend on `ecnl_verified` accuracy. ELO must note the column degradation window (May 1 → fix deployment date) in their migration spec.

---

## Section 5: If Audit Fails — Recovery Actions

### If FM2 = Path B (code fix needed, not yet applied)

Recommended minimum code change in the TGS scraper game INSERT/upsert, within the game record payload:

```javascript
ecnl_verified: (
  ['tgs', 'tgs_athleteone'].includes(source) &&
  event_tier === 'ecnl'
),
```

- Commit and deploy **before May 1**
- If deployed after May 1: supplemental backfill required for all ECNL games with `ingested_at > '2026-04-28'` and `ecnl_verified = FALSE`
- FORGE files confirmation in next briefing when deployment is confirmed
- Notify ELO of the fix date so ELO can bound the degradation window in their migration spec

### If FM2 = Path C (structural change required)

- FORGE files SENTINEL queue item (priority: urgent) within 2 hours of results received
- **Stage 2 authorization is held** until `ecnl_verified` is confirmed deployed
- May 1 pipeline launch is NOT blocked by Path C — the column defaults to FALSE, which is the pre-April-28 state
- ELO must be notified: post-May-1 ECNL games will have `ecnl_verified = FALSE` until fix is deployed; ECNL CP1 and migration planning affected

### If FM2-2 shows condition mismatch (scraper sets ecnl_verified more broadly)

- Document the specific condition difference
- Assess whether the mismatch over-flags (TRUE for non-ECNL TGS games) or under-flags (FALSE for some ECNL TGS games)
- Flag to SENTINEL; FORGE recommends aligning scraper condition to backfill condition before May 1

---

## References

- `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md` — parent audit (OUTCOME C, awaiting Presten code review)
- `Infrastructure/Pipeline Code Review Results.md` — FM2 checks Presten fills (FM2-1 through FM2-3)
- `Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md` — Presten's copy-paste grep commands
- `Infrastructure/FM1-FM2 Contingency Action Plan.md` — combined path outcomes and code fix specs
- `Infrastructure/FM1 Audit — Pre-Draft.md` — parallel audit for GA ASPIRE classifier
- `Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md` — the historical backfill this audit complements
- `Infrastructure/ECNL ecnl_verified Column — Population Plan.md` — original plan for this scraper code change
- `Infrastructure/ecnl_verified — Pipeline Write-Time Verification Spec.md` — Day 3–5 post-launch verification query

*FORGE — 2026-04-29 (pre-draft; complete within 30 minutes of Pipeline Code Review Results receipt)*
