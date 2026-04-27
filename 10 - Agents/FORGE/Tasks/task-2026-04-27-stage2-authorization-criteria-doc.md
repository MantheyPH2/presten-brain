---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Expansion — Authorization Criteria.md"
---

# Task: Stage 2 Expansion — Authorization Criteria Document

## Context

Stage 2 GotSport org-ID expansion (VA, CO, AZ states) has several pre-staging documents:
- `Stage 2 Org-ID Config Pre-Staging` — config template ready
- `Stage 2 Authorization Trigger Document` — Section 1 fill blocked until May 1–3 data
- `Stage 2 Trigger Document — Section 1 Fill` — blocked on May 1–3 monitoring data

The Stage 2 Trigger Document is designed to be filled in after Stage 1 results and May 1 data arrive. That is correct governance. But what it does not contain is the **pre-defined authorization criteria** — the specific thresholds and conditions that, once met, automatically constitute a Stage 2 GO.

Currently, SENTINEL would receive the filled trigger document and evaluate: "is this good enough to authorize Stage 2?" That evaluation requires SENTINEL to apply judgment in the moment. The better pattern: FORGE writes the criteria now, SENTINEL approves the criteria now, and the trigger document evaluation becomes a pass/fail check against known thresholds rather than a new judgment call.

This task: write the Stage 2 authorization criteria document so SENTINEL can approve the criteria before Stage 1 even runs.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Expansion — Authorization Criteria.md`

---

### Section 1: Stage 1 Completion Gate

Stage 2 cannot begin until Stage 1 TX validation is complete. Define the Stage 1 exit criteria:

**Required (all must be met):**
- Stage 1 Post-Run Validation Checklist: all 4 validation queries (V1–V4) executed
- V1 result (new game count by org-ID): **GREEN** (≥ 5,000 games per Stage 1 Expected Output Spec)
- V3 result (duplicate game check): **0 rows** (no duplicates)
- V4 result (date range sanity): games present in 2025–26 season date range

**Conditional (may proceed with caution):**
- V1 result YELLOW (500–4,999 games): FORGE files explanation with SENTINEL before Stage 2 authorization is considered. SENTINEL may authorize with reduced Stage 2 scope (one state vs. three).
- V2 result (team match rate) < 80%: FORGE files dedup analysis before Stage 2 authorization. Stage 2 expansion may exacerbate dedup issues if Stage 1 team matching is weak.

**Hard stop:**
- V1 result RED (< 500 games): Stage 2 blocked until Stage 1 is diagnosed and rerun.
- Any duplicate games introduced: Stage 2 blocked until Stage 1 dedup issue is resolved.

---

### Section 2: May 1 Pipeline Health Gate

Stage 2 adds new org-IDs to the production pipeline. The production pipeline must be confirmed healthy before adding new sources.

**Required:**
- May 1 first-run validation: PASS (FORGE must file May 1 validation results)
- 4-day game count (May 1–4): ≥ GREEN threshold (per May 2–4 Monitoring Checklist)
- No active RED signals in the pipeline as of Stage 2 authorization date

**Conditional:**
- YELLOW signals on May 3–4: FORGE must include pipeline health assessment in Stage 2 authorization request. SENTINEL may delay Stage 2 by 1 week if health is uncertain.

**Hard stop:**
- Pipeline RED (aggregate < 50 games after 48 hours): Stage 2 blocked until pipeline is diagnosed and GREEN confirmed.

---

### Section 3: Org-ID Confirmation Gate

Stage 2 expansion requires confirmed org-IDs for VA, CO, and AZ entities.

**Required:**
- Browser session completed and org-IDs confirmed for: [FORGE fills in which VA, CO, AZ entities are in scope from `va-co-az-gotsport-org-id-research.md`]
- VA, CO, AZ org-IDs added to `GotSport Org-ID Master Reference.md` with status `confirmed`
- Stage 2 config template populated with confirmed org-IDs

**Conditional:**
- If only 1 or 2 of 3 states have confirmed org-IDs: FORGE may authorize a partial Stage 2 (confirmed states only). Remaining states proceed when confirmed.

**Hard stop:**
- No browser session completed: Stage 2 blocked.

---

### Section 4: Authorization Request Format

When Stage 1, May 1, and Org-ID gates are all met, FORGE files the Stage 2 authorization request. Pre-write the template:

> **To:** SENTINEL
> **From:** FORGE
> **Re:** Stage 2 Expansion Authorization Request
>
> **Stage 1 Gate:** [PASS / CONDITIONAL — explain] — V1: [X games], V2: [X% match rate], V3: [0 / X duplicates], V4: [date range confirmed]
>
> **May 1 Gate:** [PASS / CONDITIONAL — explain] — 4-day aggregate: [X games], any RED signals: [none / describe]
>
> **Org-ID Gate:** [PASS / PARTIAL — which states] — States confirmed: [VA / CO / AZ]
>
> **FORGE recommendation:** [PROCEED with full Stage 2 / PROCEED with partial Stage 2 (states X, Y) / HOLD — explain]
>
> **Estimated Stage 2 game volume (from Stage 2 Config Pre-Staging):** [X games]
>
> **Requesting SENTINEL authorization to proceed.**

---

### Section 5: SENTINEL Pre-Approval of These Criteria

This section is left blank for SENTINEL to fill. If SENTINEL approves the criteria as written, SENTINEL adds:

> "Criteria approved — [date]. FORGE may file Stage 2 authorization request against these criteria without a new SENTINEL review."

If SENTINEL modifies any criteria, note the modification here.

---

## Quality Standard

- Section 1 thresholds must match the Stage 1 Expected Output Spec exactly. If the Expected Output Spec uses different threshold names, reconcile them — do not have two documents with inconsistent thresholds.
- Section 3 entity list must come from the actual VA/CO/AZ org-ID research document — not guessed.
- Section 4 authorization request template must be fillable in < 10 minutes by FORGE. The structure should be clear enough that FORGE is doing data entry, not writing narrative.

## Why This Matters

SENTINEL currently has no pre-written Stage 2 criteria. When FORGE eventually files the Stage 2 trigger document, SENTINEL will evaluate it on the spot. That evaluation will take longer and introduce inconsistency if SENTINEL hasn't already committed to the criteria. Writing the criteria now — before Stage 1 runs, before May 1 launches — means the Stage 2 authorization decision is already made in principle. FORGE just has to show the gates are met.

## References

- `Infrastructure/Stage 2 Expansion — Authorization Trigger Document.md`
- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md`
- `Infrastructure/Stage 1 TX Crawl — Post-Run Validation Checklist.md`
- `Infrastructure/May 1 — Pipeline Red Signal Response Protocol.md`
- `Infrastructure/Stage 2 Org-ID Config Pre-Staging.md`
- `Infrastructure/VA, CO, AZ — GotSport Org-ID Research.md`
