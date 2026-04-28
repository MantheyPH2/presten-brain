---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-03
priority: medium
status: completed
completed: 2026-04-27
topic: usl-academy-org-id-research
---

# Task: USL Academy GotSport Org-ID Research

## Context

The DSS Feature Readiness Tracker (Apr 25) identifies USL Academy (90 clubs) as a Priority 2 source gap — org IDs not confirmed as of 2026-04-24. The Source Gap Inventory lists USL Academy alongside EDP (~150+ clubs) and NAL as gaps. TX/TYSA research is already in progress (Stage 1). USL Academy is the next most tractable non-GotSport-primary source gap given it's a national league with a defined club list.

## Deliverable

File `Data Sources/USL-Academy-OrgID-Research-2026-04-27.md` in `02 - Tiger Tournaments/Projects/` with:

1. **Club list source:** Where did the list of 90 USL Academy clubs come from? Is the list current?
2. **GotSport presence check:** For a sample of 10–15 clubs, confirm whether they are in GotSport by searching `system.gotsport.com` OR by checking if games for those clubs already appear in Evo Draw. Document your sampling method.
3. **Org-ID acquisition path:** How would FORGE obtain org IDs for the full 90-club list? Options: (a) browser lookup per club, (b) GotSport API search by org name, (c) known public resource. Assess feasibility of each.
4. **Game volume estimate:** Based on the sample, how many USL Academy games per season could be added? Cross-check against the 20,000–35,000 state cup/league estimate from the DSS tracker to calibrate relative value.
5. **DSS relevance:** Is USL Academy worth prioritizing before May 9? Or is it a post-DSS expansion?
6. **Recommended next action:** Specific, actionable — e.g., "Presten should run browser session for Club X, Y, Z to confirm org IDs" or "defer to post-DSS."

## Success Criteria

- SENTINEL can decide whether to assign a USL Academy ingestion task before May 1
- Does not require FORGE to have completed all 90 org-ID lookups — a representative sample and a clear acquisition path are sufficient

## Notes

- Priority 2 means this is behind TX/TYSA (Priority 1 for DSS) — do not let this task block Stage 1 work
- If USL Academy games are already in Evo Draw via GotSport club org_ids, note that explicitly — it changes the analysis
