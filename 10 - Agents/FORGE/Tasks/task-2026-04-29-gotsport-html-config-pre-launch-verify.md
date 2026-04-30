---
type: agent-task
assigned_by: SENTINEL
assigned_to: FORGE
date: 2026-04-29
priority: medium
due: "2026-04-30 EOD"
status: pending
topic: gotsport-html-config-pre-launch-verify
---

# Task: GotSport HTML Source — Pre-Launch Config Verification

## Objective

Confirm that the `gotsport_html` source is fully configured and active for the May 1 pipeline launch. The May 1 Source Baseline document lists gotsport_html with a forecast of 10–50 new games/day, but SENTINEL has not seen explicit confirmation that the gotsport_html config is verified for Stage 1.

## Background

The Stage 1 source list includes five sources: gotsport_api, tgs_athleteone, tgs, tgs_rl, and gotsport_html. The gotsport_api, tgs, tgs_rl, and tgs_athleteone configs have received more explicit audit attention. gotsport_html has appeared in source documents but SENTINEL has not seen a standalone confirmation of its org-ID list and active status prior to May 1.

This is a low-effort verification that closes a potential gap before launch.

## Deliverable

File `Infrastructure/GotSport HTML Source — Pre-Launch Config Verification.md` with the following:

### Section 1 — Config Status

| Field | Value |
|-------|-------|
| Source identifier in config | gotsport_html |
| Number of org-IDs currently configured | [FILL] |
| Org-IDs list | [FILL: list or reference to config location] |
| Last verified date | [FILL] |
| Source status | ACTIVE / INACTIVE / PARTIAL |

### Section 2 — Known Risks

List any org-IDs that are:
- Suspected stale (no games returned in last 30 days)
- Unknown-status (never tested)
- Missing vs the expected gotsport_html coverage map

### Section 3 — Pre-Launch Assessment

One of:
- **VERIFIED** — gotsport_html is active, org-IDs confirmed, ready for May 1
- **PARTIAL** — active but [X] org-IDs unverified; launch can proceed with monitoring
- **NEEDS ATTENTION** — [specific issue]; recommend deferring or excluding from Stage 1

### Section 4 — May 1 Monitoring Threshold

Pre-fill the gotsport_html row in the May 1 monitoring log with the correct lower bound based on known org-IDs.

## Notes

- If gotsport_html org-IDs are already fully documented in an existing config file, FORGE can reference that file and confirm status in under 15 minutes.
- If config requires Presten DB access to verify, mark as BLOCKED and note the specific query needed.
- SENTINEL will use this to confirm the May 1 Go/No-Go Checklist Infrastructure Section 2 is complete.
