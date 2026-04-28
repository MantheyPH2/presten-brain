---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: pending
priority: high
due: 2026-05-06
trigger: Boys Option A verdict filed (post-April-29 queries)
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Calibration — Decision Synthesis Brief.md"
---

# Task: Boys Calibration — Decision Synthesis Brief

## Context

Boys calibration is the most complex open question in Evo Draw's ranking system. Since April 22, ELO has produced: Option A analysis framework, gap analysis, interpretation guide, calibration application specs, MLS NEXT tier split specs, U13/U14 K-factor/RD fix timing, boys ECNL scope documentation, and pre-verdict evidence checklists. Each of these lives in a separate document.

The problem: Presten and SENTINEL cannot hold all of these decisions in context simultaneously. There is no single document that answers: "Where does boys calibration stand, what decisions have been made, what is still open, and what is the timeline to resolution?"

This brief creates that document. It is not an analysis document — it is a synthesis of decisions already made or nearly made. It is the reference Presten reads before the May 9 DSS demo to understand boys ranking accuracy and what caveats apply.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Calibration — Decision Synthesis Brief.md`

File within 48 hours of Boys Option A verdict being recorded (expected: April 29 or April 30 depending on query execution).

---

### Section 1: Boys Calibration Status at a Glance

| Issue | Decision | Status | Implementation Date |
|-------|----------|--------|---------------------|
| Boys Option A (GA ASPIRE cross-gender fix) | [GO / NO-GO] | [verdict filed date] | [if GO: date; if NO-GO: N/A] |
| MLS NEXT Boys tier split calibration | Deferred | Deferred to May 18–20 | May 18 preflight begins |
| U13/U14 K-factor and RD fix | Approved, NOT YET deployed | Deploy not before May 17 | May 17 |
| Boys ECNL calibration scope | [In scope / Out of scope] | [decision date] | [if in scope: date] |
| Boys GA Aspire fix (separate from Option A) | Applied April 28 (same event_tier fix as girls) | Verification April 29 (V-series queries) | April 28 session |

ELO fills Status and dates after April 29 session.

---

### Section 2: Boys Option A — Verdict and Rationale

**Verdict:** GO / NO-GO / CONDITIONAL (fill after queries)

**Query results summary:**
- Metric 1: [fill from april29 boys query results]
- Metric 2: [fill]
- Metric 3: [fill]

**Rationale in one paragraph:** [ELO writes after verdict]

**If NO-GO:** Next evaluation window and what would need to change.

**If GO:** Implementation protocol reference: `task-2026-04-27-boys-option-a-change-execution-protocol.md`

---

### Section 3: MLS NEXT Boys — Status and Rationale for Deferral

**Current status:** Deferred to May 18–20 window.

**Why deferred (not why it should be done):**
- Boys Option A evaluation takes priority (April 29)
- MLS NEXT tier split requires stable boys baseline — cannot run concurrent with Option A implementation
- DSS window (May 9–16) does not allow MLS NEXT tier split implementation without risk of rating instability at demo time

**What May 18–20 preflight looks like:**
- Reference: `task-2026-04-24-mlsnext-calibration-application-spec.md`
- Trigger for preflight: stable post-Option-A boys ratings confirmed (1+ week of stable monitoring)

---

### Section 4: U13/U14 K-factor and RD Fix — Status

**Decision:** Approved. Scheduled for deploy not before May 17.

**Why not before May 17:**
- Pre-DSS (before May 9): too close to demo; risk of destabilizing youth age group ratings before Presten shows the product
- DSS window (May 9–16): freeze on calibration changes
- Post-DSS (May 17+): first available safe window

**Implementation readiness:** [ELO confirms which documents are ready and what remains]

---

### Section 5: Boys ECNL Calibration Scope

**Decision:** [In scope / Out of scope / Conditional]

**Rationale:** [From `task-2026-04-27-boys-ecnl-calibration-scope-documentation.md`]

**If in scope:** Timeline and dependency on ECNL migration decision (June 2).

**If out of scope:** When this decision will be revisited.

---

### Section 6: Open Boys Calibration Items (As of Filing Date)

| Item | Owner | Deadline | Blocker |
|------|-------|----------|---------|
| | | | |

ELO fills any items that remain open after Boys Option A verdict.

---

### Section 7: Boys Calibration at DSS (May 16)

**What Presten can claim at DSS:**
- [List accurate claims about boys ranking quality]

**What Presten should NOT claim:**
- [List anything that is deferred, pending, or not yet validated]

**Recommended framing:** [ELO drafts 2–3 sentences Presten can use to describe boys ranking status accurately and confidently]

---

## Definition of Done

- Filed within 48 hours of Boys Option A verdict
- Section 1 table fully populated with dates and statuses
- Section 7 (DSS framing) complete and reviewed by SENTINEL before May 9
- SENTINEL and Presten can read this document in < 5 minutes and understand the full boys calibration picture
- ELO's May 6 briefing references this document

## Why This Matters

Presten will be asked about boys ranking accuracy at DSS. There are multiple concurrent boys calibration decisions, each in a different state of completion, across 7+ documents. Without this synthesis, Presten either over-commits (claiming boys accuracy that isn't confirmed) or under-commits (discounting work that IS solid). This brief is the calibration for the pitch.

## References

- `task-2026-04-27-boys-option-a-decision-document-template.md` — verdict format (input to Section 2)
- `task-2026-04-27-boys-option-a-change-execution-protocol.md` — implementation steps if GO
- `task-2026-04-24-mlsnext-calibration-application-spec.md` — MLS NEXT spec (Section 3)
- `task-2026-04-27-boys-ecnl-calibration-scope-documentation.md` — ECNL scope (Section 5)
- `task-2026-04-27-boys-brier-pre-may17-readiness-check.md` — Brier score validation context
- `task-2026-04-28-dss-ranking-accuracy-evidence-package.md` — DSS accuracy claims context
