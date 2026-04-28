---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-09
priority: medium
status: completed
completed: 2026-04-27
topic: affinity-soccer-access-path-research
---

# Task: Affinity Soccer (IL) Access Path Research

## Context

The non-GotSport Source Priority Rerank (filed 2026-04-27) ranks Affinity Soccer (IL) at #2 post-DSS with an estimated 5,000–15,000 games/year for Illinois alone. However, the rerank explicitly notes "access path unconfirmed" and recommends scheduling an access path research sprint for June 2026 before any engineering commitment.

This task is that research sprint. It must be completed before DSS (May 16) so that if SnapSoccer deploys in May and Affinity Soccer is next, FORGE is not starting cold.

## Task

Investigate how Affinity Soccer exposes schedule and results data. Draw from publicly available information: Affinity Soccer website, visible schedule/results pages, URL patterns, any platform branding or API hints.

## Document Must Include

### Section 1: Website and Data Access
- Primary URL for Affinity Soccer schedules/results
- Are pages publicly accessible without login or registration?
- What platform or stack appears to back the site (GotSport, Demosphere, custom, etc.)?

### Section 2: Data Structure
- What does a schedule/results page look like? (HTML table, JSON API, iframe, etc.)
- Are game IDs or event IDs visible in URLs?
- Is pagination present? Approximate pages per season?

### Section 3: Geographic Scope
- Which states or leagues are covered beyond Illinois?
- Is Affinity Soccer an Illinois-only platform or multi-state?

### Section 4: Build Complexity
- Estimated HTML scraping complexity (simple table / nested structure / JS-rendered)
- Any Cloudflare, auth wall, or rate limiting observed?
- Reuse potential from existing FORGE parsers?
- Build hours estimate (range acceptable)

### Section 5: Access Path Verdict
- ACCESSIBLE (build can begin with research) / CONDITIONAL (login or contact required) / BLOCKED (no public data)
- If CONDITIONAL: what is the access path? (account registration, API key request, data export?)
- If ACCESSIBLE: revised ROI estimate (games/year ÷ build hours)

## Deliverable

File to: `Data Sources/Affinity-Soccer-Access-Path-Research-2026-04-27.md`

Update this task to `status: completed` when filed.

## Notes

This is research only — no build. The June 2026 build sprint should not be scheduled until this document confirms the access path. If the platform is GotSport or another already-supported platform, escalate immediately — that changes the priority order significantly.
