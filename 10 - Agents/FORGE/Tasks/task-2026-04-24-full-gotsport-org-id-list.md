---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: medium
due: 2026-04-28
---

# Task: Complete GotSport Org-ID List — All USYS State Associations Beyond Top 7

## Context

The GotSport org-ID expansion task (`task-2026-04-24-gotsport-org-id-expansion.md`) targeted the top 7 USYS state associations by player population: TX, CA, FL, NY, NJ, PA, WA, OH, CO (skipping IL per Affinity Soccer note). It confirmed org IDs for those states and scoped a test crawl against TX.

The gap: when Presten runs the TX test and approves production config updates, the natural next question is "which other state orgs should we add?" SENTINEL does not want to be in a position where the config update for the top 7 states requires a second research session to get the next 10 states. That research should be done now.

This task: compile the full list of GotSport org IDs for every USYS state association that uses GotSport (i.e., all except IL which uses Affinity Soccer). The goal is a production-ready config list that Presten can add all at once once the TX test confirms the approach works.

## What You Need to Do

### Step 1: Enumerate All USYS State Associations on GotSport

From your USYS State Cup Source Research and knowledge of the platform:

List all 50 states and classify each:
- **On GotSport:** has USYS state association org ID discoverable via GotSport
- **On Affinity Soccer:** uses Affinity Soccer instead (confirmed: IL; likely others)
- **On other platform:** uses a different platform (specify which)
- **Unknown:** platform not confirmed

For the states confirmed on GotSport from the prior research task (TX, CA, FL, NY, NJ, PA, WA, OH, CO): these are already documented. Copy their org IDs from `Reports/gotsport-org-expansion-test-2026-04-24.md`.

For remaining states: document what is known. If a state's GotSport org ID cannot be determined from vault knowledge, write "Org ID unknown — requires browser lookup or DB query" with the method to find it.

### Step 2: Prioritize by Player Population

Rank the states by estimated youth soccer player population (approximately: TX, CA, FL, NY, NJ, PA, WA, OH, CO, MI, MN, MA, VA, MD, NC, GA, CO, AZ, IL excluded).

For each state on GotSport, document:
| State | USYS Org Name | GotSport Org ID | Estimated player pop. | Priority |
|---|---|---|---|---|
| TX | Texas Youth Soccer Association | [ID from prior task] | ~200k | P1 |
| CA | Cal North + Cal South | [ID(s)] | ~180k | P1 |
| ... | ... | ... | ... | ... |

Mark Priority 1 (P1) for states with >50,000 estimated players, Priority 2 for 20,000–50,000, Priority 3 for <20,000.

The top-7 states already have P1 designations. The goal of this task is to complete P2 and P3.

### Step 3: Identify States That May Have Multiple Org IDs

Some large states have multiple sub-state associations each with their own GotSport org:
- **California:** Cal North Soccer (NorCal) and Cal South Soccer (SoCal) are separate organizations
- **New York:** ENYYSA (Eastern NY) and WNYSA (Western NY) are separate
- **Texas:** TYSA is the state org but many TX regional leagues have their own GotSport orgs

For each state with known sub-state splits, document all relevant org IDs. This is important for coverage completeness: a team registered under Cal South would only appear via the Cal South org ID, not Cal North.

### Step 4: Document States Using Non-GotSport Platforms

For states confirmed NOT on GotSport, document the platform and assess ingestion feasibility:

| State | Platform | Notes | Feasibility |
|---|---|---|---|
| Illinois | Affinity Soccer | [see separate research task] | TBD |
| ... | ... | ... | ... |

This gives SENTINEL a complete picture of the source gap landscape, not just the GotSport portion.

### Step 5: Produce the Final Config List

Write a production-ready config block (JSON or equivalent) listing all confirmed GotSport state association org IDs in priority order:

```json
{
  "usys_state_orgs": [
    { "state": "TX", "org_name": "Texas Youth Soccer Association", "org_id": "[ID]", "priority": 1 },
    { "state": "CA", "org_name": "Cal North Soccer", "org_id": "[ID]", "priority": 1 },
    { "state": "CA", "org_name": "Cal South Soccer", "org_id": "[ID]", "priority": 1 },
    ...
  ]
}
```

For org IDs that are confirmed from prior research: fill in the actual value.
For org IDs that are unknown: use `"org_id": null, "lookup_method": "[how to find it]"`.

The deliverable should be complete enough that Presten can paste confirmed org IDs directly into the production config without further research.

## Deliverable

Write the complete org-ID list to:
`02 - Tiger Tournaments/Projects/Data Sources/GotSport USYS Org-ID Master List.md`

Include:
- All 50 states classified (GotSport / Affinity / Other / Unknown)
- For GotSport states: org ID or lookup method, sub-state splits identified
- For non-GotSport states: platform and feasibility note
- Production-ready JSON config block

## Definition of Done

- All 50 states are classified (no "not checked" entries)
- At least 15 state org IDs are confirmed (top 7 from prior task + 8 new)
- States with sub-state splits are documented with all relevant org IDs
- Production-ready config block is included
- Summary in your briefing: how many org IDs confirmed, how many still need browser lookup

## Why This Matters

The TX test crawl is the gate before adding all state orgs to config. When Presten runs that test and it succeeds, the next action is "add all USYS state org IDs to config." Without this list, that action requires another research session. This task makes the config update a 10-minute copy-paste, not a half-day investigation.

The complete list also tells SENTINEL how much coverage uplift is available beyond the TX test — if 30 states have confirmed org IDs and they collectively represent 300,000+ teams, the coverage expansion story for DSS is materially stronger.

## References

- `02 - Tiger Tournaments/Projects/Data Sources/USYS State Cup Source Research.md` — org ID documentation for top 7 states
- `Reports/gotsport-org-expansion-test-2026-04-24.md` — confirmed org IDs (copy directly from here)
- `task-2026-04-24-affinity-soccer-source-research.md` — IL and other non-GotSport states being researched in parallel
