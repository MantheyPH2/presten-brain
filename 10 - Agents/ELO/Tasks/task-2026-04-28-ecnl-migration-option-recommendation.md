---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
due: 2026-04-30
priority: high
status: completed
completed: 2026-04-28
tags: [elo, task, ecnl, migration, recommendation, sentinel-gate, presten]
---

# Task: ECNL Migration — ELO Option Recommendation Brief

## What to Build

A decision brief that states ELO's formal recommendation on which ECNL migration option to implement. This is distinct from the option comparison matrix and the technical risk deep-dive — those are analysis documents. This brief states the verdict.

## Why This Exists

ELO has filed:
- `task-2026-04-26-ecnl-migration-option-comparison-matrix.md` — analysis
- `task-2026-04-27-ecnl-migration-option-b-technical-risk-deep-dive.md` — risk analysis
- `task-2026-04-26-ecnl-migration-option-selection-decision-gate.md` — gate criteria
- `task-2026-04-26-ecnl-migration-evidence-collection.md` — evidence

What does not exist: a brief that says "ELO recommends Option X. Here is why in three sentences. Here is the key risk to accept. SENTINEL and Presten should authorize this by April 30."

FORGE is pre-staged for all three options (`Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md`). When FORGE sees ELO's recommendation brief, it activates the chosen option's section and starts implementation. FORGE cannot act on a comparison matrix — it needs a verdict.

## Due

2026-04-30. The option decision gates both ECNL migration and FORGE's implementation timeline. Any delay past April 30 compresses FORGE's implementation window before May 9.

## Required Format

### Header

```yaml
---
type: elo-recommendation
subject: ECNL Migration Option Selection
date: 2026-04-30
recommended_option: Option A | Option B | Option C
status: recommendation-filed
sentinel_authorization_required: yes
presten_authorization_required: yes
---
```

### Recommendation Statement

> ELO recommends **[Option A / B / C — name]** for the ECNL team entity migration.

Three-sentence rationale. Each sentence addresses one factor: (1) why this option fits the current technical state, (2) why the alternatives were not selected, (3) why the timing is appropriate now.

Example structure (ELO fills in actual reasoning):
> Option [X] requires the fewest irreversible schema changes while achieving the required [outcome]. Option [Y] was rejected because [specific risk or complexity]. Option [Z] was rejected because [specific reason]. The decision is appropriate now because [timing factor].

### Key Risk Accepted

State the single most significant risk of the recommended option in one sentence. State the mitigation in one sentence. This is the risk ELO accepts, not the risks of the alternatives.

### Rating Continuity Assessment

One paragraph. For the recommended option: does ELO expect a measurable rating shift when the migration executes? YES / NO / MINIMAL (< 5 pts average). Reference: `task-2026-04-26-ecnl-migration-rating-continuity-spec.md`. If YES: state the expected range and whether it triggers the deviation threshold from that spec.

### FORGE Dependency

One sentence: which section of `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` FORGE should activate (Option A, B, or C section). State the expected FORGE implementation start date and completion window.

### Authorization Request

> ELO requests SENTINEL and Presten authorize [Option X] for implementation by [FORGE start date]. Once authorized, FORGE activates the corresponding section in the pipeline impact spec and begins implementation. SENTINEL gate: 4 conditions from Section 6 of the FORGE Pipeline Impact Spec must be met before any production schema change executes.

## Quality Standard

This document must be self-contained. Presten must be able to read it and make a YES/NO authorization decision without reading any other document. Cross-references to supporting docs are welcome; relying on them to make the case is not.

## File When Done

`02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — ELO Option Recommendation.md`

Mark this task `status: completed` in frontmatter after filing.
Notify FORGE in the same briefing session that the recommendation has been filed — FORGE monitors for this document to activate the chosen pipeline spec.
