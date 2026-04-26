---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: urgent
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/FM1-FM2 Contingency Action Plan.md"
---

# Task: FM1/FM2 Contingency Action Plan

## Context

The FM1-FM2 Quick-Run Guide is filed. Presten will run the code search before the April 28 DB session. The Pipeline Code Review Results form has three possible outcomes per check: Path A (code confirmed in vault, correct), Path B (code exists but needs fix), Path C (code not found in vault).

The two audit documents already exist and FORGE is ready to update them the moment results land. What does not exist: a pre-built action plan for each path letter combination. When Presten files results at 10 PM on April 27, FORGE needs to know immediately — without deliberation — what to build for May 1.

This task: write the contingency plan now, before results arrive. When results land, FORGE reads the plan, executes the matching branch, and files the artifacts. No decision-making under time pressure.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/FM1-FM2 Contingency Action Plan.md`

---

### Section 1: FM1 (GA ASPIRE Classifier) — Branch Table

| FM1 Result | Implication | FORGE Action | Due |
|------------|-------------|--------------|-----|
| Path A — code confirmed, classifies correctly | No change needed. May 1 deployment proceeds as planned. | Mark FM1 resolved. Update GA ASPIRE Classifier Audit with Path A outcome. | Same day results land |
| Path B — code exists, classifies incorrectly | GA ASPIRE events will continue to be misfiled after May 1 if not fixed. Code patch required before deployment. | Describe the specific patch needed: what field to modify, what logic to change, where in the scraper/classifier the fix goes. Produce a patch spec or the fix itself. | Before April 28 DB session if possible; otherwise gated on May 1 |
| Path C — code not in vault | FORGE cannot confirm the classifier exists. May 1 deployment will ingest GA ASPIRE games with unknown event_tier. | Define: does this block May 1, or proceed with monitoring? State what FORGE recommends and why. Produce a SENTINEL escalation note if blocking. | Immediate — flag before April 28 |

For Path B: FORGE must state the specific code change needed, not just "a fix is required." Reference the GA ASPIRE event_tier logic as understood from the classifier audit.

For Path C: FORGE states a concrete recommendation — hold May 1 deployment OR proceed with monitoring. No ambiguity.

---

### Section 2: FM2 (ecnl_verified Scraper Write) — Branch Table

| FM2 Result | Implication | FORGE Action | Due |
|------------|-------------|--------------|-----|
| Path A — scraper writes ecnl_verified correctly | ecnl_verified column will populate from first pipeline run. No changes needed. | Mark FM2 resolved. Update ecnl_verified Ingestion Path Audit with Path A outcome. | Same day results land |
| Path B — scraper exists but does not write the field | Column will exist after April 28 but remain all-FALSE indefinitely unless fixed. Stage 2 authorization is gated. | Produce a specific scraper patch: what write statement to add, where in the scraper code, how to verify it works. File as a separate patch spec. | Before May 1 deployment |
| Path C — scraper write not found in vault | Cannot confirm ECNL games will ever populate ecnl_verified. Stage 2 authorization blocked. | SENTINEL escalation required. FORGE files explicit block note. Propose alternative: can ecnl_verified be backfilled via SQL on a schedule? If yes, spec it. | Immediate — flag before April 28 |

For Path B: scraper patch must be specific enough for Presten to apply in a single terminal session. No abstract descriptions.

---

### Section 3: Combined Outcome Matrix

| FM1 | FM2 | May 1 Status | Stage 2 Status | FORGE Next Action |
|-----|-----|--------------|----------------|-------------------|
| A | A | ✅ Full proceed | ✅ Proceed on schedule | Update audits. File "all clear" note. |
| A | B | ✅ Proceed (monitoring) | ⚠️ Gated on scraper fix | File FM2 scraper patch. Monitor ecnl_verified population post-May 1. |
| A | C | ✅ Proceed (monitoring) | ❌ Blocked | Escalate to SENTINEL. Propose SQL backfill alternative. |
| B | A | ⚠️ Gated on FM1 fix | ✅ If FM1 fixed | File FM1 classifier patch. Get SENTINEL sign-off before April 28. |
| B | B | ⚠️ Gated on FM1 fix | ⚠️ Gated on FM2 fix | File both patches. Escalate timeline risk to SENTINEL. |
| B | C | ⚠️ Gated on FM1 fix | ❌ Blocked | File FM1 patch. Escalate FM2 block. |
| C | A | ❌ SENTINEL decision | ✅ If May 1 proceeds | Escalate FM1 C-path before April 28. |
| C | B | ❌ SENTINEL decision | ⚠️ | Escalate both. Wait SENTINEL direction. |
| C | C | ❌ SENTINEL decision | ❌ Blocked | Full escalation. SENTINEL decides May 1 hold or proceed. |

---

### Section 4: Escalation Template (C-path)

Pre-written SENTINEL escalation note. FORGE pastes this into a Queue item if any C-path is confirmed:

```
Subject: FM[1/2] Path C Confirmed — SENTINEL Decision Required
Date: [date]
From: FORGE

Presten completed the code search. FM[1/2] outcome: Path C (code not found in vault).

Implication: [describe which system is affected — classifier or scraper write]
May 1 impact: [describe whether May 1 can proceed or is blocked]
Stage 2 impact: [describe Stage 2 status]

FORGE recommendation: [hold / proceed with monitoring / proceed with alternative]
Reason: [one paragraph]

SENTINEL action required by: [date — before April 28 DB session if possible]
```

---

## Quality Standard

- Every branch must have a concrete FORGE action — not "investigate" or "TBD." If FORGE genuinely cannot know the action without seeing results, state what information is needed and who provides it.
- The Combined Outcome Matrix must cover all 9 combinations. No gaps.
- The escalation template must be copy-paste ready — FORGE fills in bracketed fields only.

## Why This Matters

Presten will file FM1/FM2 results at an unpredictable time — possibly hours before the April 28 DB session. FORGE's response window is short. A pre-built action plan eliminates the decision layer entirely: result arrives, FORGE reads the matching row, executes the action. The alternative — FORGE reasoning through implications under time pressure — risks missed details and delayed flags to SENTINEL.

## References

- `Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md` — the search guide Presten uses
- `Infrastructure/Pipeline Code Review Results.md` — where Presten files FM1/FM2 path letters
- `Infrastructure/GA ASPIRE Pipeline Classifier Audit.md` — FM1 audit document to update
- `Infrastructure/ecnl_verified Ingestion Path Audit.md` — FM2 audit document to update
- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — FM1/FM2 outcomes feed into this
