---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md"
---

# Task: April 29 Pipeline Health Check — Post-Fix Protocol

## Context

April 28 is the first major execution session: GA ASPIRE tier fix + ECNL dedup verified flag. After these two changes run and the rating recompute completes, FORGE is responsible for confirming the pipeline is healthy before normal automated operations resume.

The April 28 Reference Card covers what Presten runs during the session. This task covers what FORGE checks on April 29 (or immediately after the session) to confirm the system is clean.

Write the protocol now, before April 28, so FORGE doesn't spend session time designing the check.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md`

### Section 1: Post-GA-ASPIRE-Fix Verification

**1A: Confirm ga_aspire event_tier rows exist**
```sql
SELECT event_tier, COUNT(*) as event_count
FROM events
WHERE event_tier IS NOT NULL
GROUP BY event_tier
ORDER BY event_count DESC;
```
Expected: `ga_aspire` appears as a distinct row with count > 0. If absent, the fix did not write to the events table. Escalate to Presten immediately — do not resume pipeline until resolved.

**1B: Confirm GA ASPIRE teams are correctly tagged**
```sql
SELECT e.event_tier, COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS teams
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_tier = 'ga_aspire'
GROUP BY e.event_tier;
```
Expected: team count > 0. If 0: fix did not apply. Escalate.

**1C: Confirm no residual misclassified GA events**
Re-run whatever identification query the April 28 Reference Card uses to spot GA ASPIRE events tagged as `ga`. Expected after fix: 0 rows returned (all previously misclassified events now correctly tagged). If > 0: partial fix. Document which event IDs remain and flag to SENTINEL.

### Section 2: Post-ECNL-Dedup-Flag Verification

**2A: Confirm ecnl_verified column exists**
```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'games' AND column_name = 'ecnl_verified';
```
Expected: 1 row returned. If 0 rows: column was not added. Migration did not run. Escalate before any further ingestion.

**2B: Confirm flag was set on ECNL games**
```sql
SELECT ecnl_verified, COUNT(*) as game_count
FROM games
WHERE event_tier = 'ecnl'
GROUP BY ecnl_verified;
```
Expected: all ECNL games show `ecnl_verified = true`. If any show `false` or null: backfill did not complete. Document count and flag for Presten.

**2C: Post-fix duplicate check**
```sql
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1
ORDER BY count DESC
LIMIT 20;
```
Expected: 0 rows. If rows appear: the fix introduced or exposed duplicates. Log them in the FORGE briefing and flag to SENTINEL. Do not resume normal ingest until source is identified.

### Section 3: Rating Recompute Verification

**3A: Confirm recompute ran**
```sql
SELECT MAX(updated_at) as last_recompute
FROM rankings
WHERE age_group = 'U13G';
```
Expected: timestamp is after the April 28 session start time. If timestamp is older: recompute did not run. Ratings still reflect pre-fix state. Escalate — do NOT authorize Event Strength Phase 1 until recompute confirms.

**3B: Spot-check GA ASPIRE team rating movement**
ELO will publish the pre-fix snapshot spec at `Rankings/Pre-April-28 Baseline Snapshot Spec.md` (due April 27). After recompute, compare a known Girls GA ASPIRE team's rating against that baseline. Expected: rating increased (calibration up from `ga` level to `ga_aspire` level). Boys GA ASPIRE teams should be unchanged (both were cal=100 under Option A). If Boys moved materially (> 15 points): flag to ELO for rating shift analysis.

### Section 4: Pipeline Resume Authorization

After running Sections 1–3, state one of:

- **GREEN:** All checks pass. Normal pipeline operations authorized. FORGE resumes automated data ingest on schedule.
- **YELLOW:** Minor issues detected (specify). Automated ingest can resume with close monitoring. FORGE flags specifics to SENTINEL in next briefing.
- **RED:** Critical failure (specify). Automated ingest is NOT authorized until [specific item] is resolved. SENTINEL escalates to Presten immediately.

File the pipeline resume decision in the April 29 FORGE briefing.

## Definition of Done

- Protocol written at `Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md`
- All four sections present with executable SQL (no placeholders except the pre-fix baseline comparison in 3B, which depends on ELO's snapshot spec)
- GREEN / YELLOW / RED decision framework explicit
- Summary in briefing: "April 29 health check protocol is ready. FORGE will execute on April 29 immediately following the April 28 session."

## Why This Matters

April 28 changes two production systems (event_tier, ecnl_verified flag) and triggers a full rating recompute. Without a written post-fix check, there is no documented way to confirm the changes landed correctly before assuming the system is healthy. This protocol is the operational handoff from "Presten made the changes" to "FORGE confirms the pipeline is green."

## References

- `Product & Planning/April 28 Escalation Reference Card.md` — the April 28 session guide
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — existing validation pattern
- `10 - Agents/ELO/Tasks/task-2026-04-24-pre-april28-baseline-snapshot.md` — pre-fix snapshot (for 3B comparison)
- `10 - Agents/FORGE/Tasks/task-2026-04-23-ecnl-dedup-verified-flag-spec.md` (or equivalent) — ECNL flag specification
