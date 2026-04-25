---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md"
---

# Task: Stage 2 Expansion — SENTINEL Authorization Gate Document

## Context

Stage 2 of the GotSport org-ID expansion adds state association org IDs for USYS states (CA, FL, NY, NJ, WA, PA, OH) and any confirmed leagues from the browser session (USL Academy, EDP, potential others). The Pre-Authorization Criteria document (`Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md`) defines what must be true before Stage 2 can proceed.

What does not exist: a single SENTINEL-facing gate document that consolidates all Stage 2 authorization conditions into a checkable list. SENTINEL currently authorizes Stage 2 by reviewing 4+ documents and synthesizing across them each cycle. This creates friction and delays the authorization decision.

This task: FORGE writes the Stage 2 gate document. When all conditions in it are met, SENTINEL checks the final box and FORGE adds the Stage 2 org IDs to the crawl config. The document replaces the multi-document synthesis SENTINEL currently performs.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md`

---

### Section 1: Authorization Status Summary

A machine-readable block at the top. FORGE updates this section when any condition changes:

```
Stage 2 Authorization Status: PENDING / AUTHORIZED
Last updated: [date]
Conditions met: [N] / [total]
Blocking conditions: [list any FAIL conditions]
```

---

### Section 2: Authorization Conditions

Each condition has: source document, current status, and how FORGE confirms it.

| # | Condition | Source | Status | How to Confirm |
|---|-----------|--------|--------|----------------|
| 1 | Stage 1 TX crawl completed and Results Report filed | `Infrastructure/Stage 1 Backfill Results Report.md` | PENDING | FORGE confirms file exists with data |
| 2 | Stage 1 results show GREEN (game count ≥ threshold, no critical errors) | Stage 1 Results Report | PENDING | Results Report GREEN verdict |
| 3 | Browser session completed — USYS state org IDs confirmed (≥ 5 of 7 states) | `Infrastructure/GotSport Org-ID Master Reference.md` Section 2 | PENDING | ≥ 5 rows in Section 2 updated from pending to confirmed |
| 4 | May 1 Pre-Authorization Checklist PASS (Stage 1 is part of May 1 gate) | `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` | PENDING | Checklist filed with all items ✅ |
| 5 | Org-ID Master Reference updated with all browser-session confirmed IDs | `Infrastructure/GotSport Org-ID Master Reference.md` Section 1 | PENDING | Section 1 row count matches confirmed org IDs from browser session |
| 6 | No active YELLOW or RED pipeline errors from Stage 1 run | Morning-After Runbook results | PENDING | Morning-After Runbook filed with GREEN status |

Add any additional conditions from the Pre-Authorization Criteria doc that are not listed above. Do not remove any of the 6 conditions above without SENTINEL authorization.

---

### Section 3: Stage 2 Scope — What Gets Added

List every org-ID that enters the crawl config upon Stage 2 authorization. Pull from the Org-ID Master Reference Section 2 (confirmed) rows at the time of writing.

| Entity | Org-ID | Config Section | Condition |
|--------|--------|----------------|-----------|
| [USYS CA state assoc] | TBD — pending browser session | usys_states | Condition 3 |
| [USYS FL state assoc] | TBD | usys_states | Condition 3 |
| ... | ... | ... | ... |

Mark all TBD entries clearly. When the browser session runs, FORGE updates this table from the Org-ID Master Reference.

---

### Section 4: Authorization Trigger

When all 6 conditions in Section 2 show ✅:

1. FORGE updates Section 1 Status Summary to `AUTHORIZED`
2. FORGE files a briefing note: "Stage 2 gate cleared. Conditions 1–6 all ✅. Awaiting SENTINEL authorization to add org IDs."
3. SENTINEL reviews Section 2, confirms all ✅, and signs the authorization block below
4. FORGE adds Stage 2 org IDs to crawl config per Section 3

**SENTINEL Authorization Block** (SENTINEL fills this):

```
SENTINEL authorization: AUTHORIZED / HELD
Date: ___________
SENTINEL note: ___________
If HELD, blocking condition: ___________
```

---

### Section 5: Rollback Protocol

If Stage 2 org IDs are added and the first run produces errors:

- FORGE halts Stage 2 crawl (does not remove Stage 1 org IDs)
- FORGE identifies which Stage 2 org ID is causing the error
- FORGE removes only the problematic org ID; others remain
- FORGE reports to SENTINEL with specific error and proposed fix

Stage 2 failure does not roll back Stage 1. Stage 1 and Stage 2 are independently deployable.

---

## Quality Standard

- Section 2 conditions must be specific enough to evaluate with a yes/no. No condition can read "ensure pipeline is healthy" — it must name the specific document and the specific field to check.
- The authorization block in Section 4 must be blank when FORGE files this document. SENTINEL fills it.
- FORGE must update this document whenever a condition status changes, not just when SENTINEL requests a review.

## Why This Matters

Stage 2 authorization is currently a judgment call SENTINEL makes from memory and multi-document review. This document converts that judgment call into a checklist. The benefit to FORGE: clarity on exactly what conditions unlock Stage 2 permission, so FORGE is not waiting for SENTINEL to decide — FORGE updates the doc as conditions clear, and SENTINEL's review is confirmation rather than synthesis. The benefit to SENTINEL: one document to review, not four.

## References

- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — source conditions
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — org-ID status source
- `Infrastructure/Stage 1 Backfill Results Report.md` — Condition 1/2 source (pending execution)
- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — Condition 4 source
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — Condition 6 source
