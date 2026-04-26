---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: medium
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Product & Planning/May 9 Readiness — Gap Closure Action Plan.md"
---

# Task: May 9 Readiness — Gap Closure Action Plan

## Context

The May 9 Final Readiness Assessment Spec (filed April 26) defines the gate criteria SENTINEL uses on May 9 to determine DSS readiness. What it does not do: map each open item today (April 26) to a specific close date, responsible agent, and risk level if the close date slips.

SENTINEL currently synthesizes the May 9 path from:
- FORGE's First-Week Monitoring Log (infrastructure)
- ELO's DSS Feature Readiness Tracker (features)
- ELO's individual task files (execution windows)
- Individual briefings (status updates)

This is workable but requires SENTINEL to reconstruct the May 9 path from five documents every cycle. More importantly, it doesn't answer Presten's most important question: "What could go wrong between now and May 9, and what happens if it does?"

The Gap Closure Action Plan answers that question in one document. It is a living document — ELO updates it as items close or timelines shift. SENTINEL references it in every briefing from now through May 9.

This task is due April 28 because the plan's value is highest in the 2-week window leading up to May 9. Writing it after April 28 (when the recompute is complete) means it already reflects the first confirmed outcome.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Product & Planning/May 9 Readiness — Gap Closure Action Plan.md`

---

### Section 1: May 9 Gate Criteria (Reference)

Pull the 5–8 gate criteria from `Product & Planning/May 9 Readiness Final Assessment Spec.md` into a summary block at the top of this document. Do not rewrite them — just list them so the reader doesn't need to open a second document.

---

### Section 2: Open Items — Closure Tracking Table

One row per open item. At the time of writing (April 26–28), expected open items include:

| # | Item | Responsible | Must-Close-By | Current Status | What Blocks It | Risk if Late |
|---|------|-------------|--------------|----------------|---------------|-------------|
| 1 | Boys Option A spot check verdict | ELO | May 7 | Awaiting Presten queries (window Apr 29 – May 5) | Presten must run 3 queries | If results arrive May 5, ELO has 2 days — tight but workable |
| 2 | Girls GA ASPIRE calibration confirmed | ELO | Apr 29 (G2 gate) | Pending April 28 execution | April 28 must run and recompute complete | If April 28 fails, entire calibration track shifts |
| 3 | ECNL Migration option selection | ELO | May 10 | CP1 pending (Apr 29) | CP1 staleness check | If Option 1 disqualified May 9, Option 2 must escalate immediately |
| 4 | Calibration stability clearance | ELO | May 7 | Monitoring begins Apr 30 | Post-fix calibration behavior | If instability detected after May 5, Phase 1 may be held |
| 5 | Event Strength Phase 1 authorized | SENTINEL | May 5 | Pending G4 gate (Apr 29) and execution log | G4 must pass; execution log must confirm April 28 ran correctly | If held, Phase 1 implementation pushes; Phase 2 cannot start |
| 6 | Club Rankings — implementation start | FORGE + ELO | May 9 (start, not complete) | Pending Boys Option A decision (May 7) | Boys Option A or Girls-only fallback activation | If Boys Option A decided May 7, FORGE has 2 days before May 9 — implementable |
| 7 | USA Rank comparison result filed | ELO | May 3 | Pending Presten data collection + Apr 28 recompute | Presten must collect USA Rank data | If not done by May 5, DSS accuracy claims are unsupported |
| 8 | May 1 pipeline first-run GREEN | FORGE | May 1 | Pending pipeline deployment | FM1/FM2 path confirmed + deployment session | If RED state: FORGE + SENTINEL hold Stage 2 authorization |
| 9 | FM1/FM2 code paths confirmed | Presten | Apr 27 (optimal) | UNKNOWN — Presten must run quick-run guide | Presten action | If unknown entering May 1: deployment confidence LOW |
| 10 | Brier score completion (Girls) | ELO | May 7 | [ELO checks current status from task-2026-04-25-brier-score-completion] | Sufficient game data with corrected ratings | Low risk — Brier score is supporting evidence, not a gate blocker |

ELO fills in accurate status, blocking conditions, and risk assessments at document creation time (April 28, when more is known). Add or remove rows as needed — this is ELO's assessment of what is open, not a SENTINEL-mandated list.

---

### Section 3: Critical Path

Identify the 3 items whose delay would most directly threaten the May 9 readiness gate. Explain in 2–3 sentences why each is critical-path and what the knock-on effect of a slip looks like.

Format:

**Critical Path Item 1: [Name]**
Why critical: [mechanism]
Slip effect: [what fails downstream]

**Critical Path Item 2:** ...

**Critical Path Item 3:** ...

---

### Section 4: Risk Register

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| April 28 session delayed or partial | Low | High | All docs filed; execution log captures partial completion |
| Boys Option A queries not run by May 5 | Medium | Medium | Girls-only fallback spec ready; Club Rankings can proceed Girls-only |
| FM1/FM2 unresolved entering May 1 | Medium | Medium-High | Contingency Plan filed; Path B/C actions pre-written |
| ECNL Option 1 disqualified (CP1 fail) | Low | Medium | Option 2 escalation path defined; decision gate document ready |
| Post-fix calibration instability | Low | Medium-High | Monitoring spec filed; 3 queries running daily from Apr 30 |
| USA Rank data not collected by May 3 | Medium | Medium | Partial comparison possible; ELO flags to Presten on May 1 |

---

### Section 5: Update Protocol

ELO updates this document:
- When any row closes: mark Status = "✅ Closed [date]" and note the closing evidence
- When any timeline shifts: update Must-Close-By and note in next briefing
- When a new open item is identified: add a row

SENTINEL reviews this document in every briefing from April 29 through May 9. SENTINEL does not add rows — that is ELO's responsibility. SENTINEL may mark items "SENTINEL reviewed: [date]" in the Must-Close-By column.

---

## Quality Standard

- Section 2 table must have concrete Must-Close-By dates, not "before May 9" or "TBD"
- Risk column must distinguish between SENTINEL-controlled risks (documents, reviews) and Presten-controlled risks (running queries, browser session)
- Section 3 Critical Path must name specific items, not general categories
- The document must be updateable in < 10 minutes per session — rows are designed for single-cell updates, not paragraph rewrites

## Why This Matters

SENTINEL currently reconstructs the May 9 path in every briefing from five separate sources. That is 20–30 minutes of synthesis work per cycle that this document reduces to a 5-minute read. More importantly: if a closure date slips (Boys Option A late, FM1/FM2 unresolved, pipeline RED), SENTINEL currently discovers the problem when ELO or FORGE mentions it in a briefing. With this document, SENTINEL can see the slip coming when Must-Close-By passes without a ✅ — and escalate proactively rather than reactively.

## References

- `Product & Planning/May 9 Readiness Final Assessment Spec.md`
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md`
- `Rankings/Boys Option A — Decision Document.md`
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md`
- `Rankings/ECNL Migration Option Selection Decision Gate.md`
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md`
- `task-2026-04-26-boys-option-a-spot-check-execution`
- `task-2026-04-26-ecnl-migration-evidence-collection`
