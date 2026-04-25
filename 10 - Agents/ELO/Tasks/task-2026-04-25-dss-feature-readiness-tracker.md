---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md"
---

# Task: DSS Feature Readiness Tracker

## Context

DSS is May 16. Three features have implementation plans (Rank Bands, Event Strength Phase 1, Club Rankings). Two demo elements do not have implementation plans — they depend on execution (USA Rank comparison SQL, USYS/GotSport org coverage expansion).

What does not exist: a single document that shows all five demo elements with their current data, calibration, and implementation status. SENTINEL currently has to read across 4–5 specs to reconstruct the picture. Presten has no single document to open on May 9 and confirm what is go and what is no-go.

This task: write the tracker. ELO owns data readiness and calibration readiness. SENTINEL will update implementation status as sessions complete.

## What to Produce

Write: `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md`

---

### Section 1: Feature Readiness Matrix

For each of the 5 demo elements, assess data readiness and calibration readiness based on existing ELO work. Pull from the sources listed below — do not re-derive.

| Feature | Data Ready? | Calibration Ready? | Implementation Status | Demo Risk | Gate Requirement |
|---------|-------------|--------------------|-----------------------|-----------|-----------------|
| Rank Bands | ? | ? | Pending 5.5h session | ? | April 28 complete |
| Event Strength Phase 1 | ? | ? | Pending 7h session | ? | April 28 + GA ASPIRE stable |
| Club Rankings | ? | ? | Pending 10–11h session | ? | Boys Option A validated |
| USA Rank Comparison | ? | ? | Pending FORGE SQL (30 min) | ? | Presten psql access |
| USYS/GotSport Coverage | ? | ? | Pending Stage 1 crawl + expansion | ? | Browser session + crawl |

For each **Data Ready?** cell, ELO fills in one of:
- ✅ Ready: data is sufficient for the demo claim
- ⚠️ Conditional: ready if [specific condition] is met — state the condition explicitly
- ❌ Not ready: [specific gap] — [resolution path and estimated time]

For each **Calibration Ready?** cell, ELO fills in:
- ✅ Ready: calibration validated for this feature
- ⚠️ Pending: depends on April 28 fix landing correctly (note which features are gated on April 28)
- ❌ Known issue: [issue] — [resolution]

For **Demo Risk**: Low / Medium / High. Risk is High if the feature makes a claim that could fail visibly in front of the DSS audience.

Sources to pull from:
- Rank Bands: `Rankings/Rank Bands Threshold Validation — 2026-04-24.md`
- Event Strength: `Rankings/Event Strength Phase 1 — Full Execution Package.md` Gate A and Gate B conditions
- Club Rankings: `Rankings/Club Rankings — Implementation Specification.md` Pre-Flight Checklist
- USA Rank: `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md`, `Rankings/USA Rank Delta Interpretation Spec.md`
- USYS/GotSport: `Infrastructure/Source Gap Inventory — April 2026.md` Priority 1 and 2 items

### Section 2: Critical Path to May 9

May 9 is the final prep cutoff (one week before DSS). List every action that must complete before May 9 in chronological order. Include only actions with a downstream dependency — not nice-to-haves.

| Action | Owner | Est. Time | Gate For | Must Complete By |
|--------|-------|-----------|----------|-----------------|

Minimum expected items: April 28 session, pre-fix baseline snapshot (Presten), April 29 post-fix verification, at least one Rank Bands implementation session, FORGE running the accuracy audit SQL, system.gotsport.com browser session for org ID confirmation.

### Section 3: Fallback Coverage

The DSS Demo Spec Section 5 defines the fallback: Rank Bands + team detail + USA Rank comparison. For each feature NOT in the fallback, write one line:

"[Feature]: required for primary demo only. If [implementation session] does not complete by May 9, this feature drops to fallback demo."

This gives Presten a clean go/no-go framing: if a session misses May 9, the fallback is already documented.

### Section 4: SENTINEL Review Schedule

ELO defines the review checkpoints for this tracker:

- **April 29:** ELO updates data/calibration columns for features affected by April 28 fix. If any ⚠️ becomes ❌ post-fix, escalate to SENTINEL immediately.
- **May 9:** ELO does final readiness review. Any ❌ at this date triggers fallback for that feature. ELO files updated tracker in briefing.
- **May 14:** ELO does pre-demo check. Flags any last-minute issues to SENTINEL with action recommendation.

## Definition of Done

- Tracker written at `Product & Planning/DSS Feature Readiness Tracker — May 2026.md`
- All 5 features assessed in both Data and Calibration columns — no `?` cells remain
- Critical path table is complete, ordered, and has a named owner for each item
- Fallback coverage section is explicit for each non-fallback feature
- SENTINEL review checkpoints are dated
- Summary in briefing: "DSS Readiness Tracker filed. Data ready: [N/5]. Calibration ready: [N/5]. Critical path items: [N]. High-risk features: [list]."

## Why This Matters

SENTINEL needs to be able to answer Presten's question — "what's actually ready for DSS?" — in 30 seconds, not 20 minutes of cross-referencing specs. The tracker exists to make that answer available at any time, and to make the May 9 go/no-go review a 5-minute read rather than a session.
