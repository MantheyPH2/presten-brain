---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-05-03
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Coverage Actuals — May 2026 First Week.md"
---

# Task: Source Coverage Actuals — First Week May 2026

## Context

FORGE has filed two documents with estimated game coverage:
- `Infrastructure/Competitive Coverage Gap Inventory.md` — estimates coverage gaps vs. competitors
- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — estimates 5,000–13,000 additional games/year from 3 Tier A sources

These estimates are based on external research (source websites, event directories, platform documentation). After 3 daily pipeline runs (May 1, 2, 3), FORGE has actual ingestion data. The first-week actuals serve two purposes:

1. **Replace estimates with confirmed numbers.** The Stage 2 Authorization Trigger Document and the Coverage Gap Inventory both reference "estimated" game volumes. Post-May-1, these should read "confirmed."
2. **Identify ingestion surprises.** If a source is ingesting 60% fewer games than estimated, FORGE needs to know before SENTINEL files a DSS coverage readiness report. If a source is overperforming, that's evidence for Stage 2 expansion.

This document is also the primary input for Stage 2 Authorization Trigger Document Section 1 (due May 4). FORGE cannot populate the Section 1 performance table accurately without this coverage actuals document.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Source Coverage Actuals — May 2026 First Week.md`

---

### Section 1: First-Week Run Summary

High-level table. Run after May 3 pipeline completes (or pull from First-Week Monitoring Log):

| Date | Run Status | Total New Games Ingested | Source Errors | Notes |
|------|-----------|--------------------------|---------------|-------|
| May 1 | | | | First production run |
| May 2 | | | | |
| May 3 | | | | |
| **Totals** | | | | |

---

### Section 2: Per-Source Game Count (Active Sources Only)

For each org_id currently in the active crawl config, pull actual ingested game counts from the first 3 runs:

| Source / Org-ID | Entity Name | Estimated Annual Games | 3-Run Actual | Annualized Projection | Delta (% vs. Estimate) | Status |
|----------------|-------------|----------------------|-------------|----------------------|------------------------|--------|
| [from active config] | | | | | | ✅ On-track / ⚠️ Under / 🔴 Fail |

**Annualized projection formula:** `3-run actual × (365 / (3 × avg days between runs))`

Status key:
- ✅ **On-track:** Annualized projection within 20% of estimate
- ⚠️ **Under:** Projection 20–50% below estimate — flag in Section 4
- 🔴 **Fail:** Projection >50% below estimate OR source returned 0 games — SENTINEL escalation

---

### Section 3: Coverage Gap Inventory Update

Return to `Infrastructure/Competitive Coverage Gap Inventory.md`. For each gap that was marked "Estimated game loss: [N]," update with first-week data:

| Gap | Estimated Loss | Confirmed Status | First-Week Evidence |
|-----|---------------|-----------------|---------------------|
| [from inventory] | | ✅ Closed / ⚠️ Partial / 🔴 Confirmed open | |

A gap is "Closed" when FORGE has confirmed the source is active and ingesting games. "Partial" means the source is active but ingesting fewer games than the gap estimate implied. "Confirmed open" means FORGE cannot close the gap from the active source set.

---

### Section 4: Surprises and Flags

One paragraph per surprise:
- **Sources underperforming:** Name, estimated vs. actual, plausible cause (event calendar off-season? org_id scope too narrow?), FORGE recommendation
- **Sources overperforming:** Name, estimated vs. actual, implication for Stage 2 coverage projections
- **Sources with errors:** Name, error type, whether FORGE can self-remediate or needs Presten session

---

### Section 5: Stage 2 Authorization Input

Pull the 6 metrics that feed Stage 2 Authorization Trigger Document Section 1:

```
Daily game ingestion rate (games/day): ___
Total unique org_ids with ≥1 game ingested: ___
Pipeline uptime (runs completed / runs attempted): ___/3
Source error rate: ___% of org_ids returned errors
New teams added to database: ___
Coverage vs. competitive baseline (estimated): ___% of total addressable games
```

FORGE submits this section to `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` when filing the Section 1 fill on May 4.

---

## Quality Standard

- Section 2 must include all active org_ids — no omissions. Pull the definitive list from `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`, Section 1.
- Every "Estimated Annual Games" cell must cite its source (which document the estimate came from).
- Every Status cell must be explicit — no "TBD" or "check later."
- Section 5 numbers must be derived from Sections 1–2 data, not re-estimated.

## Why This Matters

SENTINEL cannot authorize Stage 2 expansion or file a DSS coverage readiness report on the basis of estimates alone. After May 1, SENTINEL has actual data. This document converts 3 days of pipeline runs into the evidentiary foundation for two high-stakes decisions: Stage 2 authorization (May 5) and DSS coverage claims (May 9). Writing the document format now (before May 1) means FORGE can fill it in 30 minutes after May 3's run completes, rather than designing the format under time pressure.

## References

- `Infrastructure/Competitive Coverage Gap Inventory.md` — estimates to replace
- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — Tier A estimates
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — daily run totals source
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — active source list
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — Section 1 consumer
- `task-2026-04-26-stage2-trigger-document-section1-fill` — sibling task that uses this document
