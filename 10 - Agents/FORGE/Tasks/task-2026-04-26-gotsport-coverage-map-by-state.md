---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: medium
due: 2026-05-03
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Coverage Map — States and Regions.md"
---

# Task: GotSport Coverage Map — States and Regions

## Context

The Org-ID Master Reference (filed April 27) tracks org-ID status by entity. What it does not provide is a geographic view: which US states and regions are covered by the current org-ID set, and which are absent.

SENTINEL's coverage gaps work has been source-centric (which leagues/orgs) but not geography-centric. This matters because:
1. Evo Draw should capture every game these teams play locally. Teams travel within regions, so a team ranked in Texas may play games in Oklahoma, Arkansas, or Louisiana. If those state associations aren't in the crawl, those cross-border games are missed.
2. Stage 2 expansion prioritizes orgs to add. A map view shows SENTINEL which entire states or regions have zero coverage — those are higher priority than adding a second org in a well-covered region.
3. The browser session will confirm 11 additional entities. Mapping them geographically shows whether the session closes a specific regional gap or adds density to already-covered areas.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Coverage Map — States and Regions.md`

---

### Section 1: Current Coverage — State-Level Table

For each US state that has at least one org in the current GotSport crawl config (Stage 1 + confirmed Stage 2 entities):

| State | Org(s) Present | Type (State Assoc / League / Club) | Stage | Game Volume Estimate |
|-------|---------------|-------------------------------------|-------|---------------------|
| TX | [org name] | State Assoc | Stage 1 | High |
| ... | | | | |

"Game Volume Estimate" categories: High (top state by game volume) / Medium / Low / Unknown.

Pull org-state mapping from the Org-ID Master Reference. If a state association's geographic scope isn't obvious from the org name, state it explicitly.

---

### Section 2: Coverage Gaps — States with Zero Presence

List every US state with no org in the current crawl config. Group by region (Southeast, Midwest, West, etc.) to show whether gaps are isolated or regional.

| Region | States with Zero Coverage | Notes |
|--------|--------------------------|-------|
| Southeast | [states] | |
| Midwest | [states] | |
| West | [states] | |
| Northeast | [states] | |
| South Central | [states] | |

For states that FORGE knows are covered by a league org rather than a state association (e.g., ECNL or MLS NEXT covers teams nationally), note this so SENTINEL does not flag them as gaps.

---

### Section 3: Browser Session Coverage Impact

Of the 11 entities in the Browser Lookup Session Prep Package, which states do they represent? After the browser session runs:

| Entity (from Prep Package) | State Represented | Fills a State Gap? | Priority if Confirmed |
|---------------------------|------------------|---------------------|----------------------|
| [entity 1] | | Yes / No / Partial | High / Medium / Low |
| ... | | | |

This table should be pre-populated from the prep package now and updated after the browser session results are filed.

---

### Section 4: Recommended Stage 2 Geographic Priorities

Based on the gap map, which 3–5 states represent the highest-value Stage 2 targets (high game volume states with zero or partial coverage)?

| State | Estimated Game Volume | Reason for Priority | Likely GotSport Entity to Target |
|-------|----------------------|--------------------|---------------------------------|
| | | | |

FORGE uses the non-GotSport research synthesis and any publicly available state association information to estimate which uncovered states have the highest in-scope game volume.

---

### Section 5: Coverage Summary Block

A machine-readable header (update when this document is revised):

```
Last updated: 2026-04-[date]
States with GotSport coverage: [N]
States with zero coverage: [N]
Browser session entities awaiting confirmation: [N]
States that would be newly covered if all 11 browser entities confirmed: [N]
```

---

## Definition of Done

- File written at the deliverable path
- Section 1 has a row for every state represented in current crawl config
- Section 2 identifies zero-coverage states with regional grouping
- Section 3 has pre-populated rows for all 11 browser session entities
- Section 4 names at least 3 priority states with reasoning
- Coverage Summary Block is accurate at time of writing
- Summary in briefing: "Coverage map filed. Currently covering [N] states. [N] states have zero coverage. Browser session would close [N] new states if all 11 entities confirmed. Top Stage 2 geographic priority: [state name]."

## Why This Matters

SENTINEL currently has no geographic view of coverage. Every briefing is entity-centric ("11 orgs pending browser confirmation") with no sense of whether those 11 orgs close a specific regional gap or scatter across already-covered territory. A state-level map converts the org-ID work into a coverage picture that SENTINEL can explain to Presten in one sentence: "We cover 22 states; the Southeast is our biggest geographic gap." The browser session results become geographically meaningful instead of just a list of entity names.

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — org-ID status table this map draws from
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — 11 entities to map in Section 3
- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — alternative source coverage that may partially fill state gaps
- `Infrastructure/Source Coverage Gap Inventory — April 2026.md` — league-level gaps this map complements geographically
