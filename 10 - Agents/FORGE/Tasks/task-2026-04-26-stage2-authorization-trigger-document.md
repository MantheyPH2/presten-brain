---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-05-02
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md"
---

# Task: Stage 2 Crawl Expansion — Authorization Trigger Document

## Context

Stage 2 authorization criteria exist in `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md`. The Pre-Authorization Criteria document defines what conditions must be met. What does not exist: a single unified document that (1) captures the Stage 1 results as they come in, (2) evaluates them against the criteria, and (3) produces a clear SENTINEL authorization request.

Currently, Stage 2 authorization would require SENTINEL to read the Morning-After Runbook results, cross-reference the Stage 2 criteria document, and make a judgment call. That is a synthesis step SENTINEL should not have to perform. FORGE should do the synthesis and bring SENTINEL a single yes/no question: "Stage 1 passed these criteria — authorize Stage 2?"

This document IS the synthesis. FORGE fills it out after the May 1 first run and the May 2–7 monitoring window. When it's complete, SENTINEL makes a one-look authorization decision.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md`

---

### Section 1: Stage 1 Performance Results

Fill after May 1 first run and at least 3 days of monitoring (target: file by May 5):

| Metric | Required Threshold | Actual Result | Pass? |
|--------|--------------------|---------------|-------|
| First-run game count | ≥ [N] games (pull from Validation Spec) | | ☐ |
| Run completion (no crash) | 100% | | ☐ |
| Source-level error rate | < [X]% per run | | ☐ |
| ECNL game ingestion | ≥ 1 confirmed ECNL game | | ☐ |
| GotSport org-ID coverage | All Stage 1 org-IDs responding | | ☐ |
| Duplicate rate | < [Y] duplicates per 1000 games | | ☐ |

For any metric where the threshold is "pull from [spec]," FORGE fills the Required Threshold column at document creation time. Do not leave thresholds blank — go get the numbers from the referenced specs now.

---

### Section 2: Stage 1 Blocking Issues Log

| Issue | Date Observed | Resolution | Resolved? |
|-------|---------------|------------|-----------|
| | | | |

If no issues: "No blocking issues observed in Stage 1 monitoring window (May 1–[date])."

---

### Section 3: Browser Session Org-ID Status

Before Stage 2 can expand, FORGE must confirm that browser-session org-IDs are ready to add. Pull from `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`:

| Browser Session Entity | Org-ID Confirmed? | Added to Config? | Notes |
|-----------------------|-------------------|------------------|-------|
| [list each of the 11 entities] | | | |

**Stage 2 gate:** All entities intended for Stage 2 must have confirmed org-IDs before expansion. Any entity still `pending-browser-lookup` is excluded from Stage 2 until confirmed.

---

### Section 4: Authorization Request to SENTINEL

FORGE completes this section when Sections 1–3 are fully filled:

```
Stage 1 Authorization Request
Date: [date]
From: FORGE
To: SENTINEL

Stage 1 Performance: [PASS / FAIL]
- [N] of [N] metrics passed required thresholds
- [N] blocking issues observed; all resolved / [N] unresolved (see Section 2)

Browser Session Status: [N] of 11 entities confirmed with org-IDs. [N] still pending.

Stage 2 Proposed Scope:
- Entities to add: [list of entities with confirmed org-IDs ready for Stage 2]
- Estimated new game volume: [if estimable from org-ID master reference data]
- Config change required: [Y/N] — [describe the specific config edit]

FORGE Recommendation: AUTHORIZE Stage 2 / HOLD Stage 2
Reason: [one paragraph]

SENTINEL decision requested by: [date — to hit the Stage 3 window]
```

---

### Section 5: Post-Authorization Steps (FORGE reference)

If SENTINEL authorizes Stage 2, FORGE executes in order:
1. Edit crawl config to add Stage 2 org-IDs (use `Infrastructure/GotSport Org-ID Config Edit Reference.md`)
2. Update `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — move entities from Section 2 (pending) to Section 1 (active)
3. Monitor next pipeline run for Stage 2 org-ID ingestion
4. File Stage 2 completion note in next briefing

---

## Quality Standard

- Section 1 thresholds must be specific numbers at document creation time — not "TBD" or "pull from spec later." FORGE looks them up now.
- Section 3 must list all 11 browser-session entities by name. Pull from the prep package.
- Section 4 authorization request must be fill-in-the-blank at runtime — no section should require new reasoning when FORGE fills it out post-Stage-1.
- The document must be completable in one sitting after the monitoring window closes.

## Why This Matters

SENTINEL cannot authorize Stage 2 without knowing Stage 1 passed. Currently, that knowledge lives across 5+ documents. This document consolidates the evidence and produces a one-look decision artifact. The goal: SENTINEL receives the completed Stage 2 trigger document, reads it in 3 minutes, and either signs the authorization request or flags what's missing. No multi-document cross-reference required.

## References

- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — Stage 2 gate conditions
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — Stage 1 thresholds
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — monitoring data source
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — browser session entity status
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — how to add Stage 2 org-IDs
