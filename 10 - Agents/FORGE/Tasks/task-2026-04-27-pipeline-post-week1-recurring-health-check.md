---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: open
priority: medium
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Pipeline Recurring Weekly Health Check — Post-Week-1.md"
---

# Task: Pipeline Recurring Weekly Health Check — Post-Week-1

## Context

FORGE has filed three execution-phase documents for May 1 pipeline operations:
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — deployment session
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — May 2 next-day check
- `Infrastructure/May 1 — Pipeline Incident Response Playbook.md` — failure scenarios April 29–May 3

The Morning-After Runbook defines the May 2 health check. The first-week validation period ends approximately May 7 (7 days post-launch). What is not covered: the **standing weekly health check** FORGE runs every week from May 8 onward.

After the first week, the pipeline transitions from "launch mode" (daily scrutiny) to "operating mode" (weekly monitoring). Without a recurring check protocol, SENTINEL has no systematic view of pipeline health after May 7. FORGE would monitor informally — which means monitoring inconsistently. Coverage gaps accumulate silently until a session review surfaces them, often too late for DSS.

This document defines the standing weekly protocol that persists for the life of the pipeline.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Pipeline Recurring Weekly Health Check — Post-Week-1.md`

---

### Section 1: Check Cadence and Trigger

**Frequency:** Weekly. Execute within 48 hours of each Friday pipeline run completing.

**Standing check windows:**
| Week | Target Check Date |
|------|------------------|
| Week 2 | May 9 (after May 8 or 9 run) |
| Week 3 | May 16 (before DSS demo) |
| Week 4 | May 23 |
| Ongoing | Every Friday ± 1 day |

**Emergency trigger:** Any day FORGE detects anomalous output (game count > 2× or < 50% of rolling average), run the full check immediately regardless of schedule.

---

### Section 2: Weekly Check Query Pack (5 queries, < 15 minutes)

FORGE writes actual SQL for each query. Presten runs these in a psql session; FORGE reviews outputs within one briefing cycle.

**W1 — Weekly Game Ingestion Count**
```sql
-- Returns: games ingested in the past 7 days by source
-- Compared against: prior 3-week rolling average
-- Expected: stable or increasing week-over-week
-- Flag: if < 70% of rolling average → investigation required
```

**W2 — Source Error Rate**
```sql
-- Returns: count of ingestion errors or null-source entries per source in past 7 days
-- Expected: 0 errors for any source
-- Flag: if any source shows errors → run incident response step for that source
```

**W3 — Active Org-ID Coverage**
```sql
-- Returns: org_ids with 0 games ingested in past 7 days (among org_ids in config)
-- Expected: 0 org_ids with 0 games during an active season window
-- Flag: any org_id showing 0 games during April–June → possible org_id deactivation or rate-limit
-- Exception: org_ids for leagues in off-season (no games expected) — document which leagues these are
```

**W4 — Most Recent Game Dates by Source**
```sql
-- Returns: MAX(game_date) per source
-- Expected: all sources show games within current week
-- Flag: if any source max_game_date is > 10 days old → staleness risk; check if league is in season
```

**W5 — Dedup Rate Check**
```sql
-- Returns: (new games inserted this week) / (total games processed this week) as dedup rate
-- Expected: dedup rate < 20% (most games should be new; high dedup rate may indicate duplicate ingestion)
-- Flag: if dedup rate > 40% in non-backfill weeks → investigate for duplicate org-ID entries or pipeline config issue
```

---

### Section 3: Health Assessment Framework

After running W1–W5, FORGE assigns a weekly health status:

| Status | Criteria |
|--------|----------|
| **GREEN** | W1 within 70–130% of rolling avg; W2 = 0 errors; W3 = 0 silent org-IDs in-season; W4 all sources < 10 days old; W5 dedup rate < 40% |
| **YELLOW** | 1–2 checks outside GREEN range but not RED | Flag in briefing; monitor next week |
| **RED** | W1 < 50% of rolling avg (2+ consecutive weeks); any source with 0 games 2+ consecutive weeks; W2 errors persist 2+ weeks | Escalate to SENTINEL same session |

FORGE files the weekly health status as a one-line entry at the top of each briefing:

`Pipeline health (week of [date]): GREEN | game count [N] (+X% WoW) | 0 errors | 0 silent sources`

---

### Section 4: Rolling Average Baseline Establishment

Before Week 2 (May 9) check can compare against a rolling average, FORGE needs a baseline.

After Weeks 1, 2, and 3 complete, FORGE computes:
- 3-week rolling average game ingestion (total, and per source)
- Update this section with the baseline values

Until the 3-week baseline is established (through Week 3), use the Expected Output Spec values from `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` (Stage 1 active org-IDs only), scaled up for the Stage 2 addition.

---

### Section 5: Escalation and Off-Season Handling

**Off-season handling:** Between mid-June and mid-August, many leagues are in off-season. W1 game count will naturally decline. FORGE should document the expected off-season low (approximately) and adjust GREEN thresholds accordingly. FORGE flags: "Off-season adjustment active: [start date] – [end date]. GREEN floor reduced to [N] games/week."

**SENTINEL escalation:** FORGE escalates to SENTINEL (via Queue item) when:
- RED status persists 2+ consecutive weeks
- A source that was GREEN last week is suddenly 0 games with no known reason
- The dedup rate spikes > 60% (suggests pipeline is repeatedly processing the same games — possible config loop)

**Auto-resolution:** YELLOW status for 1 week with no recurrence → resolved without escalation.

---

### Section 6: Check Log Template

FORGE uses this template to record each weekly check in the briefing:

```
## Weekly Pipeline Health — Week of [Date]

Status: GREEN / YELLOW / RED
W1 — Games ingested: [N] (rolling avg: [N], delta: [+/- %])
W2 — Source errors: [N]
W3 — Silent org-IDs in-season: [list or "none"]
W4 — Stalest source date: [source: date, age in days]
W5 — Dedup rate: [%]
Actions taken: [none / investigation note / SENTINEL escalation]
```

---

## Quality Standard

- All 5 queries (W1–W5) must be written SQL, not pseudocode placeholders. Presten runs these directly.
- GREEN/YELLOW/RED thresholds must be specific numbers. "Within normal range" is not a threshold.
- Off-season handling (Section 5) must reference specific months — do not leave this as a TBD.
- The check log template (Section 6) must be copy-paste ready for FORGE briefings.

## Why This Matters

The Morning-After Runbook closes on May 7. After that, SENTINEL's only window into pipeline health is FORGE's briefings. If FORGE monitors informally, a slow decline in source coverage (e.g., one org-ID silently deactivating) may go undetected for 2–3 weeks. By May 16 (DSS demo), SENTINEL might discover that a quarter of the game coverage has silently dropped. The weekly check query pack is a 15-minute session expense that catches that scenario in the first week it appears. The structure also converts FORGE's weekly health status from "prose in a briefing" to a machine-readable one-line flag SENTINEL can scan in 10 seconds.

## References

- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — first-week protocol this follows
- `Infrastructure/May 1 — Pipeline Incident Response Playbook.md` — incident escalation for RED triggers
- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — baseline game counts for early rolling average
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — active org-ID list for W3
