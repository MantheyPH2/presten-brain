---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Post-DSS Calibration Sprint Plan.md"
---

# Task: Post-DSS Calibration Sprint Plan

## Context

The post-DSS calibration roadmap exists (`Rankings/Post-DSS Calibration Roadmap.md`, filed today). That document describes what needs to happen after DSS. What it does not provide: an ordered, executable sprint plan with session estimates, authorization dependencies, and priority sequencing.

Presten walks into DSS (May 16) without knowing: how many sessions does calibration take after DSS? What's the order? What has to be authorized before what? What's the June 1 transition risk?

ELO should produce this answer before DSS, not after. This is a 1–2 hour planning task that converts "post-DSS roadmap" into a concrete sprint plan Presten can review.

Additionally: this plan provides ELO with a clear queue for the post-DSS period. Rather than receiving a new set of ad-hoc tasks after DSS, ELO enters post-DSS with a pre-approved sprint sequence and executes.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Post-DSS Calibration Sprint Plan.md`

---

### Section 1: Deferred Items Inventory

List every calibration item that is explicitly or implicitly deferred to post-DSS. Pull from:
- `post-dss-calibration-roadmap.md`
- Any task files with notes like "defer to post-DSS," "post-May-16," or "June 1 transition"
- SENTINEL queue items carrying open flags beyond DSS

For each item:

| Item | Why Deferred | Dependency | Estimated Sessions | DSS Impact (if deferred) |
|------|-------------|------------|-------------------|--------------------------|
| U13/U14 calibration fix | Presten decision post-April-29 | Requires post-fix stability window first | | |
| Boys Option A (if verdict delayed) | 5-day spot check window | Requires post-May-1 Boys game data | | |
| Tier 2 undefined leagues (if < 500 games) | Low volume, low DSS impact | None | | |
| Event Strength Phase 2 | Phase 1 must succeed first | Phase 1 authorization (April 29 decision) | | |
| Club rankings (Boys side, if Option A deferred) | Boys calibration must be stable first | Boys Option A verdict | | |
| Team merges high-priority audit (incomplete) | April 29 execution session | Requires Presten session | | |

ELO adds any items not listed here.

---

### Section 2: Sprint Order

Given the dependencies above, order the post-DSS calibration items into a sequenced sprint:

**Sprint 1 (May 17–21, first week post-DSS):**
- Priority: Items that are authorization-ready the moment DSS closes
- No new analysis required — just execute
- Example: If Boys Option A verdict is YES and calibration change is pre-specified, execute immediately

**Sprint 2 (May 22–28):**
- Items that depend on Sprint 1 results
- Example: Club rankings Boys side (depends on Boys calibration stability)

**Sprint 3 (June 1 window and beyond):**
- Age group transition considerations — U9/U10 roll to U10/U11, etc.
- June 1 is a hard date that affects all age-group-specific calibration

For each sprint, ELO specifies:
- What items are in scope
- Estimated session count (e.g., "2 sessions: 1 for ELO analysis, 1 for Presten authorization + execution")
- What Presten needs to do vs. what ELO can execute autonomously
- What SENTINEL authorization gates apply

---

### Section 3: June 1 Transition Risk Register

June 1 is when age groups roll over (U13→U14, U14→U15, etc.). This affects every age-group-specific calibration value. ELO filed a risk register for this (April 25). Pull from that register and state:

1. Which calibration items must be complete **before June 1** to avoid being complicated by the transition
2. Which items are safe to execute **after June 1** without transition risk
3. Whether any deferred items become significantly harder if they slip past June 1

For each high-risk item: "If this item is not complete by May 31, here is the specific complication: [describe]."

---

### Section 4: Authorization Roadmap

Some post-DSS items require Presten authorization before ELO can execute. List these explicitly with the authorization format:

| Item | Authorization Required | Suggested Authorization Timing |
|------|----------------------|-------------------------------|
| U13/U14 calibration fix | Presten confirms fix design | After April 29 gate results |
| Event Strength Phase 2 | SENTINEL authorization (Phase 1 results + Phase 2 evidence package) | After Phase 1 completes |
| Boys calibration (Option A) | Presten one-line approval | After verdict filed |
| Team merge audit entries | Presten reviews flagged merges | During Presten session |

---

### Section 5: Session Estimate Summary

Provide a total session estimate for post-DSS calibration to "fully calibrated" state:

| Phase | Estimated ELO Sessions | Estimated Presten Sessions | Target Completion |
|-------|------------------------|---------------------------|------------------|
| Sprint 1 | | | May 21 |
| Sprint 2 | | | May 28 |
| Sprint 3 (pre-June-1) | | | May 31 |
| June 1 transition | | | June 2 |
| Full calibration complete | **Total:** | **Total:** | |

This is the answer to Presten's question "how long does post-DSS calibration take?"

---

### Section 6: One-Paragraph DSS Demo Summary

Write a paragraph ELO/SENTINEL/Presten can use if the DSS audience asks: "What calibration work remains after this demo?"

The paragraph should:
- Name the deferred items honestly (without undermining confidence in the current demo)
- Give a timeframe ("fully calibrated by June 1")
- Convey that the plan exists and is underway

This is a pre-written, ready-to-use talking point.

---

## Quality Standard

- Session estimates in Section 5 must be specific numbers. "A few sessions" is not an estimate.
- Section 3 June 1 risks must name specific calibration items that slip, not just "age group transition is complex."
- Section 6 DSS summary paragraph must be a single paragraph — not a bullet list, not headers. It should read naturally as a spoken answer.

## Why This Matters

Without this document, post-DSS calibration begins reactively: Presten asks "what's next?" and ELO generates a task list in real time. With this document, the post-DSS queue is pre-defined, dependencies are mapped, and Presten can authorize the first sprint before DSS closes. ELO enters the post-DSS period with a known workplan rather than a blank slate.

## References

- `Rankings/Post-DSS Calibration Roadmap.md` (filed today — starting point)
- `Rankings/June 1 Age Transition Risk Register.md`
- `Rankings/Event Strength — Phase 2 Scope.md`
- `Rankings/Club Rankings — Boys Conditional Timeline.md`
- `Rankings/Boys Option A — Pre-Verdict Evidence Checklist.md`
- `Rankings/DSS Feature Readiness Tracker.md`
- SENTINEL 05:24 briefing — deferred items inventory in quality issues table
