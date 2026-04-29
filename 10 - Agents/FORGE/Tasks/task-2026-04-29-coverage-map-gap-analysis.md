---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: medium
due: 2026-05-02
status: pending
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 Coverage Map — Gap Analysis.md"
topic: coverage-map-gap-analysis
tags: [forge, task, coverage, source-gaps, quality]
---

# Task: Stage 1 Coverage Map — Gap Analysis

## Context

Stage 1 is launching May 1 with GotSport sources (gotsport_api, tgs_athleteone, tgs, tgs_rl) covering select org IDs. There is no single document that maps: which states and leagues are covered, which are partially covered, and which are completely absent. SENTINEL cannot assess product completeness without this map, and FORGE cannot prioritize Stage 2 org-ID expansion without knowing the gaps.

The GotSport state coverage map (task-2026-04-26-gotsport-coverage-map-by-state) exists. This task synthesizes it into a gap analysis tied to the [[League Hierarchy]] — which leagues need coverage, which leagues have it.

## What to Build

Produce: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 Coverage Map — Gap Analysis.md`

---

## Required Sections

### Section 1 — Methodology

Briefly state: How "coverage" is defined for this document. Coverage means: at least one org ID in Stage 1 config that runs games for teams in this league within this state.

### Section 2 — League × State Coverage Matrix

Build a matrix using the top leagues from [[League Hierarchy]] (ECNL, MLS NEXT, NPL, USL Academy, EDP, GA, GA ASPIRE, State Cups, regional cups) × the states where Evo Draw has meaningful team density.

| League | States Covered (Stage 1) | States Partial | States Not Covered | Coverage % |
|--------|--------------------------|---------------|-------------------|------------|
| ECNL | | | | |
| MLS NEXT | | | | |
| NPL | | | | |
| ... | | | | |

"Covered" = org IDs in config that are confirmed to capture this league in this state.
"Partial" = some org IDs present but known gaps (e.g., state has 10 clubs, only 3 org IDs loaded).
"Not covered" = no org IDs active for this league in this state.

Fill from the existing gotsport coverage map and org-ID config. Acknowledge uncertainty where data is incomplete.

### Section 3 — Top Coverage Gaps (Priority Ranked)

List the 5–10 most significant gaps. Each gap entry:

| Rank | League | State | Estimated Teams Missing | Source Needed | Priority |
|------|--------|-------|------------------------|---------------|---------|
| 1 | | | | | |

Prioritize by: team density × league calibration weight × data availability.

### Section 4 — Non-GotSport League Coverage

Leagues that don't use GotSport (e.g., SnapSoccer-based, SincSports-based, direct export):

| League | Source Type | Current Status | Estimated Launch |
|--------|------------|----------------|-----------------|
| USL Academy | SincSports | Design filed | May-June (pending auth) |
| SnapSoccer leagues | SnapSoccer | Pre-check filed | Pending authorization |
| ... | | | |

Reference `Infrastructure/Non-GotSport Source Priority Recommendation.md` for the ranked list.

### Section 5 — Stage 2 Expansion Recommendation

Top 3 GotSport org-ID additions that would have the highest coverage impact:
1. [State/Region] [League] — [org IDs needed] — [estimated impact: X teams, Y games/week]
2.
3.

Reference the gotsport-coverage-map-by-state and existing org-ID research.

---

## Definition of Done

- Section 2 matrix covers all top-10 leagues from League Hierarchy
- Section 3 lists at least 5 gaps with priority rankings
- Section 4 covers all known non-GotSport sources
- Section 5 has concrete Stage 2 recommendations
- Document is filed and referenced in next FORGE briefing

## References

- `task-2026-04-26-gotsport-coverage-map-by-state.md` — state coverage data
- `Infrastructure/Non-GotSport Source Priority Recommendation.md` — non-GotSport sources
- [[League Hierarchy]] — leagues to map
- `task-2026-04-24-source-gap-inventory.md` — existing gap inventory
