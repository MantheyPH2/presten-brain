---
title: May 9 DSS — Calibration Readiness Brief
tags: [dss, calibration, rankings, may9, sentinel-gate, elo]
created: 2026-04-28
author: ELO
status: template — data fills May 1–8; SENTINEL authorization May 8–9
---

# May 9 DSS — Calibration Readiness Brief

> **Purpose:** Answers "How confident are you that these rankings are accurate?" for the May 9 DSS pre-flight. Filed as a template April 28; data fills happen May 1–8; SENTINEL reviews and authorizes by May 8–9.

---

## Section 1: Calibration Status at May 8

*ELO fills from Pre-May-9 Calibration State Dashboard. Do not pre-fill with placeholders.*

| Metric | Girls | Boys | Source |
|--------|-------|------|--------|
| Brier score (target < 0.23) | _[fill May 8]_ | _[fill May 8]_ | Pre-May-9 Calibration State Dashboard |
| USARank comparison (% top-100 within 3 positions) | _[fill]_ | _[fill]_ | `Rankings/USA Rank Post-April-28 Comparison — Execution Package.md` |
| Event Strength Phase 1 status | _[Active / Not yet active]_ | N/A | This brief |
| GA ASPIRE fix applied and verified | _[YES / NO]_ | N/A | April 29 gate results |
| Boys Option A calibration | _[Applied / Not applied]_ | — | April 29–May 5 verdict |
| Tier 2 calibration fix | _[Applied]_ | _[Applied]_ | April 27 deployment confirmation |

---

## Section 2: What We Got Right (Confidence Builders)

*ELO fills from actual data at time of filing. Do not pre-fill with placeholders.*

_[3–5 bullet points with specific numbers once April 29 gates and May 1–7 monitoring complete]_

Examples of what to include when data is available:
- Girls Brier score improvement from pre-GA-ASPIRE-fix to post-fix
- USARank comparison agreement rate for top-100 teams
- League calibration verification against reference match set

---

## Section 3: Known Limitations (Honest Disclosure)

| Limitation | Severity | Explanation |
|------------|----------|-------------|
| Boys calibration (Option A not applied if verdict pending) | Medium | "Boys model is strong but boys-specific calibration is provisional through May 17" |
| MLS NEXT Boys tier split | Medium | "Deferred to May 18–20 post-DSS sprint" |
| Team merges: 27,820 entries, partial audit | Low-Medium | "Bulk of merges are correct; high-priority audit complete by May 10" |
| ECNL migration (June 2) | Low | "Not a DSS-day concern; implementation plan ready" |

*This section is for ELO's honest internal use; SENTINEL decides what Presten needs to communicate.*

---

## Section 4: Competitive Context

*ELO fills one paragraph by May 8, comparing Evo Draw ranking accuracy to nearest competitor.*
*Source: `Competitors` vault note.*

_[Fill May 1–8 from competitive analysis. Key question: meaningfully better, comparable, or behind on ranking accuracy as of May 9?]_

---

## Section 5: SENTINEL Authorization Statement

**SENTINEL fills this section after reviewing Sections 1–4.**

```
SENTINEL DSS Calibration Readiness: AUTHORIZED / CONDITIONAL / NOT YET AUTHORIZED

Authorization date: 2026-05-0_
Conditions (if any):

Signature: SENTINEL
```

---

## Section 6: What Changes This Brief

ELO updates this brief if any of the following happen before May 8:

| Event | Impact on Brief |
|-------|----------------|
| Boys Option A calibration applied | Update Section 1 + Section 3 (limitation removed or downgraded) |
| Event Strength Phase 1 activated | Update Section 1 + Section 2 (confidence builder added) |
| USARank comparison results filed | Update Section 1 + Section 2 |
| Team merges audit Presten results in | Update Section 3 (severity may reduce) |

ELO flags updates to SENTINEL with each briefing after May 1.

---

*ELO — template filed 2026-04-28 | data fills May 1–8 | SENTINEL review May 8–9*

## References

- `Rankings/Pre-May-9 Calibration State Dashboard.md` — data source for Section 1
- `Rankings/DSS Demo Talking Points — Rankings Features.md` — incorporate into Section 2
- `Rankings/DSS Competitive Accuracy Narrative.md` — competitive context for Section 4
- `Rankings/DSS Demo Validation Criteria — May 2026.md` — DSS overall criteria
