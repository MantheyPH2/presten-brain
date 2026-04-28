---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-09
priority: medium
status: completed
completed: 2026-04-27
topic: nal-platform-identification-research
---

# Task: NAL Platform Identification Research

## Context

The non-GotSport Source Priority Rerank (filed 2026-04-27) ranks NAL at #3 with "Platform ID required first" and "Unknown" for games/year, build hours, and ROI. Before any engineering commitment can be made, FORGE must identify what platform NAL uses for game scheduling and results. Without this, NAL cannot be planned into any post-DSS sprint.

## Task

Research what platform(s) the National Academy League (NAL) uses for game management, schedule publication, and results reporting. Draw from publicly available information — NAL website, visible URL patterns, platform branding, any accessible schedule or results pages.

## Document Must Include

### Section 1: Platform Identity
- What platform does NAL use? (GotSport, Demosphere, Blue Star, custom, etc.)
- How confident is the identification? (confirmed via URL / inferred from HTML / uncertain)
- If GotSport: what is the org_id or org_id range? Is it already in the GotSport Org-ID Master Reference?

### Section 2: Data Accessibility
- Are schedule/results pages publicly accessible without login?
- What URL structure is used for game data?
- Any rate limiting, Cloudflare, or auth wall observed?

### Section 3: Coverage Estimate
- How many teams/age groups does NAL cover?
- Which states or regions?
- Estimated games/year if data is accessible?

### Section 4: Build Path
- If GotSport: config add only (5 min), no build required — state the org_id
- If non-GotSport: estimated build hours and parser approach
- Any reuse from existing FORGE parsers?

### Section 5: Recommendation
- GO (config add or build with clear path) / HOLD (needs more research) / NO-GO (inaccessible or too low volume)
- If GO: proposed placement in post-DSS engineering queue relative to SnapSoccer and Affinity Soccer

## Deliverable

File to: `Data Sources/NAL-Platform-Identification-Research-2026-04-27.md`

Update this task to `status: completed` when filed.

## Notes

This is a research task only — no build work. Output is a platform identification + GO/NO-GO recommendation. If NAL turns out to be a GotSport org, this task immediately resolves to a config add and upgrades NAL ahead of all non-GotSport sources in priority.
