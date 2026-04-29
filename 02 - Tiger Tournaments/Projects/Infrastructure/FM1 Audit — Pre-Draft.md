---
title: FM1 Audit — GA ASPIRE Pipeline Classifier (Pre-Draft)
tags: [infrastructure, pipeline, ga-aspire, classifier, audit, fm1, forge]
created: 2026-04-29
updated: 2026-04-29
author: FORGE
status: pre-draft — awaiting Pipeline Code Review Results from Presten
task: task-2026-04-28-fm1-fm2-audit-pre-draft
---

# FM1 Audit — GA ASPIRE Pipeline Classifier

> **STATUS: PRE-DRAFT — Awaiting Pipeline Code Review Results from Presten**
> Fill [FILL: ...] placeholders, remove this header, and move to final path `Infrastructure/FM1 Audit.md` on receipt.
>
> **Time to complete on receipt:** < 30 minutes (static sections are pre-populated; only result cells and verdict require Presten's data).

---

## Section 1: What FM1 Does

**FM1 = GA ASPIRE event_tier classifier** — the logic in the GotSport crawler that assigns `event_tier = 'ga_aspire'` to incoming game records.

| Field | Value |
|-------|-------|
| Scope | GotSport crawler event tier classification |
| Expected location | `crawlers/gotsport/classify-event.js` or equivalent — or an inline tier-mapping object within the main scraper module |
| Trigger | Executes at ingest time when a GotSport game record is normalized before INSERT |
| Field assigned | `event_tier` on the `games` table |
| Correct behavior | GA ASPIRE events must receive `event_tier = 'ga_aspire'`, not `event_tier = 'ga'` |

**Why this matters:** The April 28 session applies a one-time `UPDATE games SET event_tier = 'ga_aspire' WHERE ...` to fix all historical records. If the pipeline classifier does not sustain this fix going forward, every new GA ASPIRE game ingested after April 28 will land at `event_tier = 'ga'` and the fix erodes daily.

**Reference audit document:** `Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md`

---

## Section 2: What the Code Review Is Checking

FM1 checks (from `Infrastructure/Pipeline Code Review Results.md`):

| Check ID | What Presten Runs | What Correct Behavior Looks Like |
|----------|-------------------|----------------------------------|
| FM1-1 | `grep -rn "ga_aspire" .` (codebase root) | Returns at least one file:line in the classifier — not just a comment or test fixture |
| FM1-2 | Locate the event_tier assignment block; check rule order | `ga_aspire` rule appears **before** the generic `ga` fallback |
| FM1-3 | Check the game INSERT/upsert for `ON CONFLICT DO UPDATE SET event_tier` | If present, re-scraping would overwrite the April 28 fix — this is a separate risk to document |

**Pattern that must match at minimum:**
```javascript
// GA ASPIRE rule — must appear before generic GA fallback
if (
  eventName.includes('GA ASPIRE') ||
  eventName.includes('GA-ASPIRE') ||
  /\bASPIRE\b/i.test(eventName)
) {
  return 'ga_aspire';
}
```

---

## Section 3: Result Table

| Check | Presten's Finding | File:Line | Status |
|-------|-------------------|-----------|--------|
| FM1-1: `ga_aspire` string found in codebase | [FILL: from Pipeline Code Review Results] | [FILL: file:line] | [FILL: FOUND / NOT FOUND] |
| FM1-2: Rule order confirmed (before generic GA) | [FILL: from Pipeline Code Review Results] | [FILL: file:line] | [FILL: CONFIRMED / NOT CONFIRMED] |
| FM1-3: `ON CONFLICT DO UPDATE SET event_tier` check | [FILL: from Pipeline Code Review Results] | [FILL: file:line or N/A] | [FILL: PRESENT / NOT PRESENT] |
| FM1 Path determined | [FILL: A / B / C] | — | — |

**FM1 Path definitions:**
- **Path A:** `ga_aspire` rule found, correctly ordered, no overwrite risk → no code change needed
- **Path B:** `ga_aspire` absent or misplaced → simple lookup addition required (≤ 5 lines)
- **Path C:** Classifier architecture prevents simple fix → structural change required, SENTINEL escalation

---

## Section 4: Verdict

**FM1 Path: [FILL: A / B / C]**

| Verdict Criterion | Result |
|-------------------|--------|
| Classifier sustains April 28 fix going forward | [FILL: YES / NO / UNKNOWN] |
| Code change required before May 1 | [FILL: YES (Path B/C) / NO (Path A)] |
| Re-scrape overwrite risk identified | [FILL: YES / NO] |
| Audit status after this filing | [FILL: CLOSED (Path A) / CONDITIONAL CLOSED (Path B fix confirmed) / ESCALATED (Path C)] |

**Pass criteria:** FM1 = Path A, OR FM1 = Path B with fix confirmed committed and deployed before May 1.
**Fail criteria:** FM1 = Path C, OR FM1 = Path B fix NOT deployed before May 1 pipeline launch.

---

## Section 5: If Audit Fails — Recovery Actions

### If FM1 = Path B (code fix needed, not yet applied)

Recommended minimum code change to `classify-event.js` (or equivalent), before the generic `ga` fallback:

```javascript
if (
  eventName.includes('GA ASPIRE') ||
  eventName.includes('GA-ASPIRE') ||
  /\bASPIRE\b/i.test(eventName)
) {
  return 'ga_aspire';
}
```

- Commit and deploy **before May 1**
- If deployed after May 1: supplemental `UPDATE games SET event_tier = 'ga_aspire' WHERE source = 'gotsport' AND event_tier = 'ga' AND [date range after April 28]` required
- FORGE files confirmation in next briefing when deployment is confirmed

### If FM1 = Path C (structural change required)

- FORGE files SENTINEL queue item (priority: urgent) within 2 hours of results received
- May 1 deployment is NOT blocked by Path C (GA ASPIRE classification is an accuracy issue, not a launch blocker) — but SENTINEL must be informed before launch
- ELO must be notified: GA ASPIRE rating accuracy post-May 1 may be degraded until fix is deployed

### If FM1-3 shows overwrite risk (ON CONFLICT includes event_tier)

- Document the specific upsert SQL
- FORGE recommends removing `event_tier` from the ON CONFLICT SET clause for GotSport records
- Flag to SENTINEL as a separate item (may be addressed in a post-May-1 patch window)

---

## References

- `Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md` — parent audit (OUTCOME C, awaiting Presten code review)
- `Infrastructure/Pipeline Code Review Results.md` — FM1 checks Presten fills (FM1-1 through FM1-3)
- `Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md` — Presten's copy-paste grep commands
- `Infrastructure/FM1-FM2 Contingency Action Plan.md` — combined path outcomes and code fix specs
- `Infrastructure/FM2 Audit — Pre-Draft.md` — parallel audit for ecnl_verified write path
- `Infrastructure/April 28 Execution Log.md` — confirms whether April 28 DB fix applied (context for this audit)

*FORGE — 2026-04-29 (pre-draft; complete within 30 minutes of Pipeline Code Review Results receipt)*
