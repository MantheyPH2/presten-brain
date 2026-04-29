---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: in-progress
priority: high
due: 2026-05-08
deliverable: "02 - Tiger Tournaments/Projects/Rankings/May 9 DSS — Calibration Readiness Brief.md"
---

# Task: May 9 DSS — Calibration Readiness Brief

## Context

On May 9, Presten has a DSS demo. SENTINEL must authorize readiness. The existing pre-May-9 calibration state dashboard (`task-2026-04-28-pre-may9-calibration-state-dashboard.md`) is a data table — it will show current Brier scores, percentages, and query results. What it is NOT is a narrative brief that answers the question Presten will be asked in the room:

**"How confident are you that these rankings are accurate?"**

Presten needs a clear, honest, one-page narrative answer before walking into DSS. This brief provides it. ELO files it by May 8, SENTINEL reviews and signs off (or flags concerns) before the May 9 pre-flight.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/May 9 DSS — Calibration Readiness Brief.md`

---

### Section 1: Calibration Status at May 8 (ELO fills from dashboard)

| Metric | Girls | Boys | Source |
|--------|-------|------|--------|
| Brier score (lower = better; target < 0.23) | | | Pre-May-9 dashboard |
| USArank comparison (% top-100 within 3 positions) | | | `Rankings/USArank Comparison Results.md` |
| Event Strength Phase 1 status | Active / Not yet active | N/A | This brief |
| GA ASPIRE fix applied and verified | YES / NO | N/A | April 29 gate results |
| Boys Option A calibration | Applied / Not applied | — | April 29–May 5 verdict |
| Tier 2 calibration fix | Applied | Applied | April 27 deployment |

---

### Section 2: What We Got Right (Confidence Builders)

3–5 bullet points ELO can speak to in the demo room. Examples:

- "Girls Brier score improved from [X] to [Y] after GA ASPIRE fix — quantified accuracy improvement"
- "USArank comparison shows [Z]% of top-100 teams within 3 ranking positions"
- "[League name] calibration verified against [N] reference matches with expected spread"

ELO fills from actual data at time of filing. Do not pre-fill with placeholders in this section.

---

### Section 3: Known Limitations (Honest Disclosure)

What ELO would flag if asked "what's still imperfect?":

| Limitation | Severity | Explanation |
|------------|----------|-------------|
| Boys calibration (Option A not applied if verdict pending) | Medium | "Boys model is strong but boys-specific calibration is provisional through May 17" |
| MLS NEXT Boys tier split | Medium | "Deferred to May 18–20 post-DSS sprint" |
| Team merges: 27,820 entries, partial audit | Low-Medium | "Bulk of merges are correct; 5–10% require manual review by May 10" |
| ECNL migration (June 2) | Low | "Not a DSS-day concern; implementation plan ready" |

This section is for ELO's honest internal use, not necessarily verbatim for Presten. SENTINEL reviews and decides what Presten needs to know.

---

### Section 4: Competitive Context

ELO fills one paragraph comparing Evo Draw ranking accuracy to the nearest competitor. Source: `Competitors` vault note. Key question: Are we meaningfully better, comparable, or behind on ranking accuracy as of May 9?

This gives Presten a positioning statement if a DSS attendee asks "why is Evo Draw better than [competitor]?"

---

### Section 5: SENTINEL Authorization Statement

**SENTINEL fills this section after reviewing Sections 1–4.**

```
SENTINEL DSS Calibration Readiness: AUTHORIZED / CONDITIONAL / NOT YET AUTHORIZED

Authorization date: 2026-05-0_
Conditions (if any):

Signature: SENTINEL
```

---

### Section 6: What Changes This Brief

ELO updates this brief if any of the following happen before May 8:

| Event | Impact on Brief |
|-------|----------------|
| Boys Option A calibration applied | Update Section 1 + Section 3 (limitation removed) |
| Event Strength Phase 1 activated | Update Section 1 + Section 2 (confidence builder added) |
| USArank comparison results filed | Update Section 1 + Section 2 |
| Team merges audit Presten results in | Update Section 3 (severity may reduce) |

ELO flags updates to SENTINEL with each briefing after May 1.

---

## Definition of Done

- Template filed at deliverable path by April 30 (Sections 1 headers + 3–6 structure pre-written; data fills happen May 1–8)
- All data fields in Section 1 populated by May 8
- Sections 2 and 3 written in narrative form (not placeholders) by May 8
- SENTINEL reviews and fills Section 5 authorization block on May 8 or 9
- ELO's May 8 briefing references this document as ready for SENTINEL review

## Why This Matters

The pre-May-9 calibration dashboard tells Presten WHAT the numbers are. This brief tells Presten WHAT TO SAY about them. DSS is a pitch environment — a table of Brier scores with no narrative is not a demo asset. The combination of dashboard (data) + this brief (narrative) gives Presten everything needed to answer the calibration credibility question confidently.

SENTINEL's authorization block in Section 5 also creates a formal record that SENTINEL reviewed and approved the calibration state before DSS. If questions arise afterward, this document is the audit trail.

## References

- `task-2026-04-28-pre-may9-calibration-state-dashboard.md` — data source for Section 1
- `task-2026-04-27-dss-demo-talking-points-rankings.md` — existing talking points (incorporate into Section 2)
- `task-2026-04-27-dss-competitive-accuracy-narrative.md` — competitive context draft (check if Section 4 is already covered)
- `task-2026-04-25-dss-demo-spec.md` — DSS demo overall spec
- `Competitors` vault note — competitive comparison data
