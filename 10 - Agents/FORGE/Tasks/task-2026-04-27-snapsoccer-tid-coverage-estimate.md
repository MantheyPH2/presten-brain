---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-05
priority: medium
status: completed
completed: 2026-04-27
topic: snapsoccer-tid-coverage-estimate
---

# Task: SnapSoccer TID Coverage Estimate

## Context

FORGE filed the SnapSoccer Minimum Viable Parser Spec on 2026-04-27. The spec correctly identifies the entry point as `snapsoccer.com/events/` for TID discovery. However, two gaps remain before the May 17 build session:

1. **Unknown TID count:** The total number of active TIDs on SnapSoccer has not been estimated. Without this, FORGE cannot predict the ingestion scope, the time required for TID discovery, or the expected game volume.

2. **Expected game volume unset:** The parser spec targets post-DSS build. Before SENTINEL can include SnapSoccer in the post-DSS sprint plan, FORGE needs to provide a game volume estimate so it can be compared against the 34,961 known SincSports games (bulk import, not live scraped). The question: does SnapSoccer represent a meaningful new volume, or mostly overlap with the existing SincSports dataset?

This task produces the missing estimate so SENTINEL and Presten can set realistic expectations for the May 17 session.

## Task

Produce a `Data Sources/SnapSoccer-TID-Coverage-Estimate-2026-04-27.md` document covering:

### Section 1: TID Inventory Estimate

Based on available evidence (the SnapSoccer events page structure, known tournament count if documented, analogous platform sizes), estimate:
- Approximate number of unique active TIDs (tournaments/leagues with live or recent schedules)
- Breakdown if possible: tournaments vs. leagues vs. recreational events
- Confidence level: LOW / MEDIUM / HIGH — and explain why

If the events page is not accessible without a browser check, note this explicitly and provide a range estimate based on what is known about SincSports tournament volume.

### Section 2: Expected New Game Volume

Cross-reference the TID estimate with the following:
- Known SincSports game count: 34,961 (bulk import, historical)
- That bulk import was a one-time file import, not live scraping — SnapSoccer likely has more recent events not captured in the bulk import
- Estimate: of the total SnapSoccer game volume, what fraction is likely NOT already in the database?

Produce a range (low / mid / high scenario) for net new games the SnapSoccer ingestion would add.

### Section 3: Discovery Runtime Estimate

Using the rate-limit spec from the parser (800–1000ms between requests), and the TID count estimate:
- How long would a full TID discovery crawl take?
- Is a full crawl needed at every run, or can TIDs be cached with a weekly refresh?
- Recommend: one-time full discovery + nightly delta (check for new events) vs. full discovery every run

### Section 4: Pre-Build Session Recommendation

Based on sections 1–3, provide a go/no-go recommendation for including SnapSoccer in the May 17 build session:
- **GO:** TID count is tractable, expected new volume is meaningful, discovery runtime is acceptable
- **CONDITIONAL GO:** One condition must be confirmed first (state it)
- **DEFER:** Reason why May 17 is too early or scope is unclear

## Deliverable

File to: `Data Sources/SnapSoccer-TID-Coverage-Estimate-2026-04-27.md`

Update this task to `status: completed` when filed.

## Notes

This estimate is advisory. The browser check (`Data Sources/SincSports-Browser-Check-Protocol-2026-04-27.md`) must still confirm ACCESSIBLE before the build session begins. But the TID coverage estimate is independent of the browser check — FORGE can produce it before Presten runs the browser check.

Due May 5 — needs to be in SENTINEL's hands before the post-DSS sprint planning conversation.
