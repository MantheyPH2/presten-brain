---
type: agent-task
assigned_to: FORGE
assigned_by: CHIEF
date: 2026-04-24
status: in_progress
priority: high
due: 2026-04-28
vault_deliverable: complete
code_implementation: pending-presten
notes: All script specs fully documented in Daily Pipeline Cadence Scope.md (Section 6). Backoff fix must stabilize first before this goes live.
---

# Task: Implement Option A — Daily Pipeline with Active Team Filter

## Context

You scoped three pipeline cadence options and recommended Option A (Smart Overnight Batch with Active Team Filter). The scoping is done. Implementation is the next step. DSS is May 16 — you have until May 1 to have this running and tested.

## What to Build

Following your own spec from `02 - Tiger Tournaments/Projects/Product & Planning/Daily Pipeline Cadence Scope.md`:

1. **Implement `get-active-team-ids.sh`** — the SQL query that identifies teams with games in the last 90 days or scheduled in the next 30 days. Output: comma-separated ID list for `GSAPI_TEAM_IDS`.

2. **Implement `run-daily.sh`** — the daily script that:
   - Calls `get-active-team-ids.sh` to build the active team ID list
   - Runs the crawl against active IDs only (~15,000–40,000 teams vs 107,528)
   - Runs the full pipeline after crawl completes
   - Logs success/failure with timing

3. **Configure cron:**
   - Daily run: `0 2 * * 1-6` (Monday–Saturday, 2 AM)
   - Weekly full sweep: `0 23 * * 0` (Sunday 11 PM — `run-overnight.sh`, unchanged)

4. **Implement health check** — script that detects if a run failed (checks for completed log marker, alerts if missing)

## Testing Requirement

Before setting the cron live:
- Run `run-daily.sh` manually once in a test window
- Confirm active team ID count is in the expected 15,000–40,000 range
- Confirm crawl completes in under 6 hours
- Confirm pipeline produces updated rankings
- Document results in your briefing

## Deliverable

- All scripts written and deployed
- Cron entries configured
- Test run completed and documented in your 2026-04-28 briefing (or sooner)
- Rollback plan confirmed: `run-overnight.sh` still functional as fallback

## References

- `02 - Tiger Tournaments/Projects/Product & Planning/Daily Pipeline Cadence Scope.md` — your own spec, everything you need is in here
- `run-overnight.sh` — existing overnight batch (template for structure)
