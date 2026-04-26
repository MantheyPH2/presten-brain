---
title: Source Coverage Actuals — May 2026 First Week
tags: [infrastructure, pipeline, coverage, actuals, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: template — FORGE fills after May 1–3 pipeline runs
---

# Source Coverage Actuals — May 2026 First Week

> **FORGE:** Fill this document after 3 pipeline runs complete (May 1, 2, 3). Target filing: end of day May 3. Section 5 feeds `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` Section 1 (due May 4). Do not estimate — use actual pipeline output from `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md`.

---

## Section 1: First-Week Run Summary

*(Pull from First-Week Pipeline Monitoring Log — May 1–7.md after May 3 run completes)*

| Date | Run Status | Total New Games Ingested | Source Errors | Notes |
|------|-----------|--------------------------|---------------|-------|
| May 1 | | | | First production run |
| May 2 | | | | |
| May 3 | | | | |
| **3-Run Totals** | | | | |
| **3-Run Average** | | | | |

---

## Section 2: Per-Source Game Count (Active Sources Only)

*(Pull active org-ID list from `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` Section 1. Pull estimates from `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` and `Infrastructure/Competitive Coverage Gap Inventory.md`.)*

| Source / Org-ID | Entity Name | Estimated Annual Games (source) | 3-Run Total | Annualized Projection¹ | Delta vs. Estimate | Status |
|----------------|-------------|--------------------------------|------------|----------------------|-------------------|--------|
| | | | | | | |
| | | | | | | |
| | | | | | | |
| **Total (all active sources)** | | | | | | |

¹ Annualized projection = `3-run total × (365 ÷ (3 × avg days between runs))`

**Status key:**
- ✅ **On-track:** Annualized projection within 20% of estimate
- ⚠️ **Under:** Projection 20–50% below estimate — explain in Section 4
- 🔴 **Fail:** Projection >50% below estimate OR source returned 0 games — SENTINEL escalation required

---

## Section 3: Coverage Gap Inventory Update

*(Reference `Infrastructure/Competitive Coverage Gap Inventory.md`. For each gap previously marked with an estimated game loss, update with first-week evidence.)*

| Gap | Prior Estimated Loss | Confirmed Status | First-Week Evidence |
|-----|---------------------|-----------------|---------------------|
| | | ✅ Closed / ⚠️ Partial / 🔴 Confirmed open | |
| | | ✅ Closed / ⚠️ Partial / 🔴 Confirmed open | |
| | | ✅ Closed / ⚠️ Partial / 🔴 Confirmed open | |

**Status definitions:**
- **Closed:** Source is active and ingesting games at or above estimate.
- **Partial:** Source is active but ingesting fewer games than the gap estimate implied.
- **Confirmed open:** Source not ingesting or not yet active — gap not closed by current pipeline scope.

---

## Section 4: Surprises and Flags

*(One paragraph per surprise. Write only entries that exist — delete unused headings.)*

### Sources Underperforming

*(Name, estimated vs. actual, plausible cause — off-season scheduling gap? Org-ID scope too narrow? — and FORGE recommendation.)*

### Sources Overperforming

*(Name, estimated vs. actual, implication for Stage 2 coverage projections.)*

### Sources with Errors

*(Name, error type, whether FORGE can self-remediate or requires a Presten session.)*

---

## Section 5: Stage 2 Authorization Input

*(Fill after 3 runs. Submit to `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` Section 1 on May 4.)*

```
Snapshot date: 2026-05-03 (after May 3 run)
Derived from: First-Week Pipeline Monitoring Log, Source Ingestion Baseline

Daily game ingestion rate (games/day):              ___
Total unique org_ids with ≥1 game ingested:         ___ of ___ active org_ids
Pipeline uptime (runs completed / runs attempted):  ___ / 3
Source error rate (% of org_ids returning errors):  ___%
New teams added to database (net new):              ___
Coverage vs. competitive baseline (estimated):      ___% of total addressable games
```

**FORGE assessment for Stage 2 Authorization Trigger Document Section 4:**
*(Complete only after filling all fields above.)*

```
Based on 3-run actuals:
- [N/6] Stage 2 performance metrics PASS required thresholds
- Blocking issues: [none / list any Severity 3 events]
- FORGE recommendation: AUTHORIZE Stage 2 / HOLD Stage 2
- Reason: [one paragraph — cite specific metrics]
```

---

## References

- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — run totals source
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — active source list for Section 2
- `Infrastructure/Competitive Coverage Gap Inventory.md` — gap estimates for Section 3
- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — Tier A source estimates
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — Section 5 feeds this
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — comparison baseline
- `task-2026-04-26-stage2-trigger-document-section1-fill` — sibling task consuming Section 5

*FORGE — 2026-04-26*
