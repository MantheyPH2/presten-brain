---
title: FM1-FM2 Contingency Action Plan
tags: [infrastructure, pipeline, ga-aspire, ecnl, contingency, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — activate when Presten files Pipeline Code Review Results
task: task-2026-04-26-fm1-fm2-contingency-action-plan
---

# FM1-FM2 Contingency Action Plan

> **How to use:** When Presten files FM1/FM2 path letters in `Infrastructure/Pipeline Code Review Results.md`, FORGE reads this plan, executes the matching branch, and files artifacts. No deliberation under time pressure.

---

## Section 1: FM1 (GA ASPIRE Classifier) — Branch Table

| FM1 Result | Implication | FORGE Action | Due |
|------------|-------------|--------------|-----|
| **Path A — code confirmed, classifies correctly** | No change needed. `ga_aspire` is in the classifier map and fires before the generic GA fallback. April 28 DB fix is sustained. May 1 proceeds as planned. | Mark FM1 resolved. Update GA ASPIRE Classifier Audit frontmatter: `status: outcome-A-cleared`, add `outcome_confirmed: [date]` and `fm1_path: A`. Add confirmed rule location to Section 3. | Same day results land |
| **Path B — code exists but classifies incorrectly** | Every new GA ASPIRE game ingested after May 1 will have `event_tier = 'ga'`. The April 28 DB fix erodes with each new scrape. Code patch required **before May 1 deployment**. | 1. Update GA ASPIRE Classifier Audit with Path B outcome (see update protocol task). 2. Apply the code fix below. 3. Add to May 1 Pre-Auth Checklist Section 3: "GA ASPIRE classifier fix committed and deployed: YES / NO." 4. Notify SENTINEL that May 1 is gated on this fix being confirmed deployed. | Before April 28 DB session if possible; otherwise **required before May 1** |
| **Path C — code not found in vault** | FORGE cannot confirm the classifier exists. May 1 deployment will ingest GA ASPIRE games with unknown `event_tier`. | 1. File SENTINEL escalation using the template in Section 4 (FM1 C-path). 2. FORGE recommendation: proceed with April 28 DB session; hold May 1 deployment pending SENTINEL direction. 3. Open new task: "GA ASPIRE Classifier — Pre-May-1 Code Fix" for SENTINEL to scope. | Immediate — escalate before April 28 DB session |

### FM1 Path B — Specific Code Fix

Add the following rule in the event tier classifier, **before the generic `ga` fallback**:

```javascript
// In event_tier classification logic — add before generic GA fallback
if (
  eventName.includes('GA ASPIRE') ||
  eventName.includes('GA-ASPIRE') ||
  /\bASPIRE\b/i.test(eventName)
) {
  return 'ga_aspire';
}
```

This addition is ≤ 5 lines. Target file: wherever `event_tier` is assigned during GotSport game normalization (expected: `crawlers/gotsport/classify-event.js` or equivalent — confirm from FM1-1/FM1-2 notes).

**Verification after applying the fix:**

```bash
grep -n "ga_aspire" [classifier-file-path]
```

Expected: the new rule appears, positioned before any generic `ga` fallback. Then run a dry-run crawl against a known GA ASPIRE event and confirm `event_tier = 'ga_aspire'` in the output.

**FM1-3 note:** If Presten confirmed `event_tier` IS in an `ON CONFLICT DO UPDATE` clause (FM1-3 = YES), the classifier fix must be deployed and confirmed **before any re-scrape of existing GA ASPIRE games**. A re-scrape before the fix would overwrite the April 28 UPDATE with the wrong tier again.

---

## Section 2: FM2 (ecnl_verified Scraper Write) — Branch Table

| FM2 Result | Implication | FORGE Action | Due |
|------------|-------------|--------------|-----|
| **Path A — scraper already writes ecnl_verified correctly** | Column will populate correctly from the first pipeline run. No code change needed. | Mark FM2 resolved. Update ecnl_verified Ingestion Path Audit frontmatter: `status: outcome-A-cleared`, add `outcome_confirmed: [date]` and `fm2_path: A`. Add confirming sentence to Section 4. | Same day results land |
| **Path B — scraper exists but does not write the field** | Column will exist after April 28 but remain all-FALSE indefinitely unless fixed. Every ECNL game ingested post-May-1 will have `ecnl_verified = FALSE`. Stage 2 authorization NOT blocked, but scraper fix must be applied with or before May 1 launch. | 1. Update ecnl_verified Ingestion Path Audit with Path B outcome. 2. Apply the scraper patch below. 3. Add condition to May 1 Pre-Auth Checklist Section 3: "ecnl_verified scraper code change confirmed applied: YES / NO." 4. If fix cannot be deployed before May 1: note that post-May-1 ECNL games will need a supplemental backfill. | Before May 1 deployment |
| **Path C — scraper architecture prevents simple field addition** | Column cannot be populated by simple addition. All new ECNL games from May 1 forward have `ecnl_verified = FALSE` indefinitely. ELO migration spec assumptions are affected. **Stage 2 authorization is held.** | 1. File SENTINEL escalation using Section 4 template (FM2 C-path). 2. Propose SQL-based alternative (see note below). 3. Update ecnl_verified Audit with Path C outcome. 4. Notify ELO that ecnl_verified accuracy is gated on architecture fix. | Immediate — flag before April 28 |

### FM2 Path B — Specific Scraper Patch

Add the following field to the TGS scraper game INSERT/upsert payload (target file from FM2-1 notes):

```javascript
// In TGS scraper game write logic — add to INSERT payload
ecnl_verified: (
  ['tgs', 'tgs_athleteone'].includes(source) &&
  event_tier === 'ecnl'
),
```

This is a ≤ 5-line addition. Confirm the variable names (`source`, `event_tier`) match the scraper's actual field names for game records — use the FM2-1 file:line as the reference point.

**Verification after applying the fix:**

```bash
grep -n "ecnl_verified" [tgs-scraper-file-path]
```

Expected: assignment found. Then on the first pipeline run after May 1, run:

```sql
SELECT COUNT(*) FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND created_at > NOW() - INTERVAL '24 hours'
  AND ecnl_verified = TRUE;
```

Expected: count > 0 (new ECNL games have `ecnl_verified = TRUE`).

### FM2 Path C — SQL Alternative (Fallback if architecture blocks simple addition)

If the scraper cannot be patched, propose a scheduled SQL approach to SENTINEL:

```sql
-- Nightly catch-up UPDATE — run after each pipeline cycle
UPDATE games
SET ecnl_verified = TRUE
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND ecnl_verified = FALSE
  AND created_at > NOW() - INTERVAL '48 hours';
```

This can be appended to `run-daily.sh` as a post-crawl step. Limitation: any gap between pipeline run and catch-up UPDATE leaves games temporarily inaccurate. Flag this limitation explicitly in the SENTINEL escalation.

---

## Section 3: Combined Outcome Matrix

| FM1 | FM2 | May 1 Status | Stage 2 Status | FORGE Next Action |
|-----|-----|--------------|----------------|-------------------|
| A | A | ✅ Full proceed | ✅ Proceed on schedule | Update both audits to Path A. File "all clear" in next briefing. |
| A | B | ✅ Proceed (with condition) | ✅ Proceed (monitor ecnl_verified) | File FM2 scraper patch. Add FM2 condition to May 1 Pre-Auth Checklist. Monitor ecnl_verified population after May 1 first run. |
| A | C | ✅ Proceed (with monitoring) | ❌ Blocked until scraper fix confirmed | Escalate FM2 to SENTINEL. Propose SQL alternative. Stage 2 authorization held. Notify ELO. |
| B | A | ⚠️ Gated on FM1 classifier fix | ✅ If FM1 fixed before May 1 | Apply FM1 code fix. Get SENTINEL confirmation before proceeding. May 1 proceeds only after fix confirmed deployed. |
| B | B | ⚠️ Gated on FM1 fix | ✅ If FM1 fixed before May 1 (monitor ecnl_verified) | Apply both fixes. Escalate timeline risk to SENTINEL if both cannot be deployed before May 1. |
| B | C | ⚠️ Gated on FM1 fix | ❌ Blocked until FM2 fix | Apply FM1 fix. Escalate FM2 block to SENTINEL. Wait for SENTINEL direction on Stage 2. |
| C | A | ❌ SENTINEL decision required | ✅ If May 1 proceeds | Escalate FM1 C-path before April 28. April 28 DB session proceeds. May 1 gated on SENTINEL direction. |
| C | B | ❌ SENTINEL decision on May 1 | ✅ If May 1 proceeds (with FM2 fix) | Escalate FM1 C-path. Apply FM2 scraper patch (unblocked from May 1 decision). |
| C | C | ❌ SENTINEL decision required | ❌ Blocked | Full escalation. Both paths C. SENTINEL decides May 1 hold or conditional proceed. Stage 2 blocked until FM2 resolved. |

**Decision authority:** FM1 Path B requires SENTINEL sign-off before April 28. FM1 Path C blocks May 1 (SENTINEL decides). FM2 Path C blocks Stage 2. All other paths are FORGE-executable without SENTINEL pause.

---

## Section 4: Escalation Templates (C-path)

### FM1 Path C Escalation

```
Queue Item — FORGE to SENTINEL
Type: alert
Priority: urgent
Date: [date of FM1 results]

Subject: FM1 Path C Confirmed — May 1 Deployment Decision Required

Presten completed the FM1 code search. Result: Path C — `ga_aspire` classifier logic not found in vault/codebase.

Implication: FORGE cannot confirm whether the GotSport crawler assigns event_tier = 'ga_aspire' to GA ASPIRE events. If no classifier rule exists, every GA ASPIRE game ingested after May 1 will land at event_tier = 'ga'. The April 28 DB fix will erode with each new scrape.

May 1 impact: FORGE recommends holding May 1 deployment pending scoping of the classifier fix. Alternatively, proceed with May 1 and monitor GA ASPIRE event_tier assignments — but every day without the fix risks data degradation.

FORGE recommendation: HOLD May 1 pending code fix scoped and confirmed deployable. Fix is ≤ 5 lines if the classifier follows the expected lookup-dict pattern.

SENTINEL action required by: [date — before April 28 DB session if possible, or before May 1 at latest]
```

### FM2 Path C Escalation

```
Queue Item — FORGE to SENTINEL
Type: alert
Priority: urgent
Date: [date of FM2 results]

Subject: FM2 Path C Confirmed — Stage 2 Authorization Blocked; ecnl_verified Fix Required

Presten completed the FM2 code search. Result: Path C — `ecnl_verified` write cannot be added via simple field addition.

Implication: The TGS scraper architecture prevents a simple `ecnl_verified: true` addition to the INSERT payload. All new ECNL games after May 1 will have ecnl_verified = FALSE indefinitely unless an architecture fix or alternative approach is implemented.

May 1 impact: May 1 deployment can proceed — the column exists and historical backfill is complete. Forward ingestion accuracy is degraded but not a deployment blocker.

Stage 2 impact: Stage 2 authorization is held. FORGE will not recommend Stage 2 expansion while ecnl_verified ingestion is broken.

FORGE recommendation: Authorize May 1 with monitoring condition. Scope the ecnl_verified fix (or SQL catch-up alternative) as a pre-Stage-2 requirement. ELO must be notified that ecnl_verified accuracy is limited to pre-April-28 games until fix is deployed.

SENTINEL action required by: before Stage 2 authorization window (May 5)
```

---

## References

- `Infrastructure/Pipeline Code Review Results.md` — FM1/FM2 path letters (trigger for this plan)
- `Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md` — FM1 update target
- `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md` — FM2 update target
- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — conditions added for Path B
- `Infrastructure/April 28 Reference Card.md` — context for FM1-3 ON CONFLICT risk
- `task-2026-04-26-audit-documents-outcome-update-protocol.md` — update protocol for the audit files

*FORGE — 2026-04-26*
