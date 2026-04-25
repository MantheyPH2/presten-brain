---
type: protocol
author: FORGE
date: 2026-04-24
status: ready
applies_after: April 28 execution session (GA ASPIRE tier fix + ECNL dedup verified flag)
---

# April 29 Pipeline Health Check — Post-Fix Protocol

Run this protocol on April 29 (or immediately after the April 28 session completes) to confirm the two production changes landed correctly and the pipeline is safe to resume.

**Context:** April 28 changes two production systems (`event_tier` reclassification for GA ASPIRE, `ecnl_verified` flag on ECNL games) and triggers a full rating recompute. This protocol is the operational handoff from "Presten made the changes" to "FORGE confirms the pipeline is green."

---

## Section 1: Post-GA-ASPIRE-Fix Verification

### 1A: Confirm `ga_aspire` event_tier rows exist

```sql
SELECT event_tier, COUNT(*) AS event_count
FROM events
WHERE event_tier IS NOT NULL
GROUP BY event_tier
ORDER BY event_count DESC;
```

**Expected:** `ga_aspire` appears as a distinct row with `event_count > 0`.
**If absent:** Fix did not write to the events table. Escalate to Presten immediately — do not resume pipeline until resolved.

---

### 1B: Confirm GA ASPIRE teams are correctly tagged

```sql
SELECT e.event_tier,
       COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS teams
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_tier = 'ga_aspire'
GROUP BY e.event_tier;
```

**Expected:** `teams > 0`.
**If 0:** Fix did not apply. Escalate.

---

### 1C: Confirm no residual misclassified GA events

Run the same identification query used in the April 28 Reference Card to spot GA ASPIRE events previously tagged as `ga`.

```sql
SELECT COUNT(*) AS remaining_misclassified
FROM events
WHERE event_tier = 'ga'
  AND (event_name ILIKE '%aspire%' OR event_name ILIKE '%ga aspire%');
```

**Expected after fix:** `0 rows` — all previously misclassified events now correctly tagged.
**If > 0:** Partial fix. Document which event IDs remain and flag to SENTINEL.

---

## Section 2: Post-ECNL-Dedup-Flag Verification

### 2A: Confirm `ecnl_verified` column exists

```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'games' AND column_name = 'ecnl_verified';
```

**Expected:** 1 row returned.
**If 0 rows:** Column was not added. Migration did not run. Escalate before any further ingestion.

---

### 2B: Confirm flag was set on ECNL games

```sql
SELECT ecnl_verified, COUNT(*) AS game_count
FROM games
WHERE event_tier = 'ecnl'
GROUP BY ecnl_verified;
```

**Expected:** All ECNL games show `ecnl_verified = true`.
**If any show `false` or null:** Backfill did not complete. Document count and flag for Presten.

---

### 2C: Post-fix duplicate check

```sql
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1
ORDER BY count DESC
LIMIT 20;
```

**Expected:** 0 rows.
**If rows appear:** Fix introduced or exposed duplicates. Log them in the FORGE briefing and flag to SENTINEL. Do not resume normal ingest until source is identified.

---

## Section 3: Rating Recompute Verification

### 3A: Confirm recompute ran

```sql
SELECT MAX(updated_at) AS last_recompute
FROM rankings
WHERE age_group = 'U13G';
```

**Expected:** Timestamp is after the April 28 session start time.
**If older:** Recompute did not run. Ratings still reflect pre-fix state. Escalate — do NOT authorize Event Strength Phase 1 until recompute confirms.

---

### 3B: Spot-check GA ASPIRE team rating movement

ELO will publish the pre-fix snapshot at `Rankings/Pre-April-28 Baseline Snapshot Spec.md` (due April 27). After recompute, compare a known Girls GA ASPIRE team's rating against that baseline.

**Expected:** Rating increased (calibration up from `ga` level to `ga_aspire` level).
**Boys GA ASPIRE teams:** Should be unchanged (both at cal=100 under Option A). If Boys moved materially (> 15 points): flag to ELO for rating shift analysis.

---

## Section 4: Pipeline Resume Authorization

After running Sections 1–3, FORGE states one of the following in the April 29 briefing:

**GREEN:** All checks pass. Normal pipeline operations authorized. FORGE resumes automated data ingest on schedule.

**YELLOW:** Minor issues detected (specify). Automated ingest can resume with close monitoring. FORGE flags specifics to SENTINEL in next briefing.

**RED:** Critical failure (specify). Automated ingest is NOT authorized until [specific item] is resolved. SENTINEL escalates to Presten immediately.

---

## Quick Reference Checklist

```
□ 1A: ga_aspire event_tier row exists (count > 0)
□ 1B: GA ASPIRE team count > 0
□ 1C: Remaining misclassified GA events = 0
□ 2A: ecnl_verified column confirmed in schema
□ 2B: All ECNL games show ecnl_verified = true
□ 2C: Duplicate check = 0 rows
□ 3A: Rankings.updated_at is after April 28 session start
□ 3B: Girls GA ASPIRE rating increased vs pre-fix baseline
□ Pipeline resume decision: GREEN / YELLOW / RED (file in briefing)
```

---

## References

- `Product & Planning/April 28 Escalation Reference Card.md` — the April 28 session guide
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — existing validation pattern
- `Rankings/Pre-April-28 Baseline Snapshot Spec.md` — ELO's pre-fix snapshot (for 3B comparison, due April 27)
