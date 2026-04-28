---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: pending
priority: urgent
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/TYSA Config Add — Execution Checklist.md"
---

# Task: TYSA Config Add — Execution Checklist

## Context

The May 1 Stage 1 Config Final Review card (filed today) is pre-certified except for Section 5, which requires the TYSA org-ID. The EDP + USL Academy Config Activation Card is ready. The browser session to look up TYSA's org-ID at `system.gotsport.com` is the only remaining action before FORGE can certify and May 1 deployment is fully authorized.

The problem: when Presten completes the browser session and returns the org-ID, there must be zero lag between receipt and config add. Every minute of delay is a minute less before the May 1 run. The Config Activation Card describes WHAT to activate. This task produces the HOW — the specific step-by-step commands FORGE executes the moment the org-ID arrives.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/TYSA Config Add — Execution Checklist.md`

---

### Section 1: Pre-Receipt Readiness (Complete Now)

FORGE confirms these are true before the browser session:

- [ ] Config file location confirmed: `[file path]`
- [ ] Config syntax format documented: `org_id: [value]` or `"org_id": "[value]"` or other
- [ ] Test org-ID (any known working org-ID) verified as valid syntax in the config
- [ ] Dry-run command available: `[command to validate config without running]`
- [ ] First-run monitoring query written and ready: `SELECT COUNT(*) FROM games WHERE source = 'TYSA' AND played_at >= NOW() - INTERVAL '7 days';`

**Pre-receipt status:** READY / NOT READY

If NOT READY: document what is missing. Do not wait for the org-ID to resolve this.

---

### Section 2: Execution Steps (Complete Upon Org-ID Receipt)

Execute in order. Do not skip steps.

**Step 1 — Add TYSA to Config (< 5 min)**
```
1. Open config file at: [path]
2. Add entry: org_id = [TYSA org-ID from browser session], state = TX, name = "TYSA"
3. Verify no trailing comma / syntax errors
4. Save file
```

**Step 2 — Syntax Validation (< 2 min)**
```
[dry-run or lint command]
Expected output: "Config valid" or equivalent — no errors
```
If validation fails: stop, document the error, file SENTINEL queue item before retrying.

**Step 3 — Confirm Config Written Correctly (< 1 min)**
```
[command to read back the TYSA entry from the config]
Expected: org_id matches exactly what was provided by Presten
```

**Step 4 — Sign May 1 Stage 1 Config Final Review (Section 5)**
```
Open: Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md
Section 5: FORGE CERTIFICATION
  - Status: CERTIFIED
  - Date signed: [today's date]
  - TYSA org-ID confirmed: [value]
  - Config syntax verified: YES
  - FORGE signature: FORGE — [date and time]
```

**Step 5 — Notify via Briefing**

FORGE's next briefing includes:
- TYSA org-ID received: [value]
- Config add: COMPLETE
- May 1 certification: SIGNED
- First-run monitoring: ready for May 1

---

### Section 3: First-Run Validation (Execute After May 1 Pipeline Runs)

```sql
-- TYSA game ingestion check (run after first post-May-1 pipeline run)
SELECT 
  COUNT(*) as total_games,
  MIN(played_at) as earliest_game,
  MAX(played_at) as latest_game
FROM games 
WHERE source_org_id = '[TYSA org-ID]';
```

Expected result: 3,500–7,000 games; dates span current season.

If result < 1,000 games: flag immediately. Either the org-ID is wrong or TYSA has minimal game history in GotSport.

---

## Definition of Done

- Section 1 (Pre-Receipt Readiness) completed and status = READY before browser session
- Sections 2–3 executed within 30 minutes of org-ID receipt
- May 1 Stage 1 Config Final Review Section 5 signed
- First-run validation query filed and run after May 1 pipeline executes
- Results documented in FORGE's May 1 or May 2 briefing

## Why This Matters

The TYSA org-ID is the last unresolved item before May 1 deployment is fully authorized. Presten should be able to hand FORGE the org-ID and have the config certified within 30 minutes. This checklist eliminates any execution ambiguity and ensures FORGE can sign off immediately.

## References

- `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` — requires Section 5 signature
- `Infrastructure/EDP + USL Academy Config Activation Card.md` — parallel activation (same session)
- `task-2026-04-26-stage2-authorization-trigger-document.md` — Stage 2 gates on May 1 Stage 1 certification
- `task-2026-04-27-browser-session-post-receipt-action-plan.md` — general post-receipt protocol; this doc is TYSA-specific
