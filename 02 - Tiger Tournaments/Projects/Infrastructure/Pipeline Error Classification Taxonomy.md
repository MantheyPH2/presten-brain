---
title: Pipeline Error Classification Taxonomy
tags: [infrastructure, pipeline, errors, monitoring, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: active — use in all briefings from May 1 forward
---

# Pipeline Error Classification Taxonomy

> Standard vocabulary for all FORGE pipeline error reporting. Every briefing and health log from May 1 onward uses these codes and formats. SENTINEL reads: "0 Tier 1, 1 Tier 2, 4 Tier 3" and immediately knows severity.

---

## Section 1: Error Tier Definitions

### Tier 1 — Critical (pipeline outcome compromised)

**Definition:** Any error that results in (a) zero games ingested for a full run, (b) corrupted or malformed data written to the DB, or (c) pipeline process crash before completion.

**Required action:** Flag to SENTINEL in the same-session briefing. Do NOT file GREEN status for any run with a Tier 1 error. Include `CRITICAL:` prefix in briefing error summary. Do NOT proceed to Stage 2 authorization while any Tier 1 error is unresolved.

**Examples:**
- Pipeline exits with non-zero status code before completing Step 9
- `get-active-team-ids.sh` returns empty set → N=0 teams crawled
- Database write failure (INSERT or UPDATE returns error, row not persisted)
- Cron job did not fire (no `DAILY PIPELINE STARTED` in log)
- Entire run returns 0 new games on a day when games are expected (see `RUN-EMPTY` below)

---

### Tier 2 — Degraded (partial outcome, recoverable)

**Definition:** Errors that reduce game ingestion below expected threshold for one or more sources, but do not corrupt existing data and do not crash the pipeline process. Pipeline completes; output is incomplete.

**Required action:** Log in briefing with specific source and error type. Escalate to SENTINEL if the same Tier 2 error appears on 2 or more consecutive runs (48+ hours). Include `[T2]` tag in briefing error line.

**Examples:**
- One or more GotSport org-IDs returning rate-limit responses (429) after backoff exhausted
- Source fetch timeout for specific org-ID (pipeline completes but that source contributes 0 games)
- Parsing failure causing partial game record — row rejected by DB constraint, not written
- `run-daily.sh` completes but Step 2 (dedup) produces anomalously high merge count

---

### Tier 3 — Noise (routine, expected, self-resolving)

**Definition:** Errors that occur at low frequency during normal operation, do not affect overall game count or data integrity, and are expected artifacts of the crawl at scale.

**Required action:** Log count only in briefing error summary. No escalation unless count exceeds **25 per run** (anomalous noise floor — investigate on 2+ consecutive runs).

**Examples:**
- GotSport returns 404 for an inactive org-ID or team that has no current-season schedule
- Transient HTTP timeout on a single team fetch, retried successfully by backoff logic
- Score field in raw payload is null (game not yet played) — excluded from DB write correctly
- Empty payload for a team with no games in the current season window

---

## Section 2: Error Type Codes

Format: `[SOURCE/LAYER]-[TYPE]`

| Code | Description | Tier | Expected Frequency |
|------|-------------|------|--------------------|
| `GS-RATELIMIT` | GotSport returned 429 after all backoff retries exhausted | T2 | Low — backoff should prevent most; flag if > 3 per run |
| `GS-TIMEOUT` | GotSport org-ID or team endpoint timed out (no response within threshold) | T2 | Low (isolated); Medium if widespread |
| `GS-404` | GotSport returned 404 — org-ID or team not found (likely inactive) | T3 | Medium (inactive orgs/teams are expected) |
| `GS-EMPTY` | GotSport returned valid 200 but payload contains zero games | T3 | Medium (off-season teams, teams with no schedule) |
| `GS-MALFORMED` | GotSport response could not be parsed — unexpected JSON structure | T2 | Very low; flag immediately if > 1 per run |
| `DLQ-ENTRY` | Team fetch exhausted all backoff retries — placed in dead-letter queue | T2 | Low; monitor counts vs prior runs |
| `PARSE-SCORE` | Score field missing, null, or non-numeric in otherwise valid game record | T3 | Very low |
| `PARSE-DATE` | Game date missing or could not be parsed — record excluded | T3 | Very low |
| `PARSE-TEAM` | Home or away team_id missing or unresolvable — record excluded | T2 | Very low; if > 5 per run investigate classifier |
| `DB-WRITE` | Database INSERT or UPDATE returned an error — game not persisted | T1 | Should be zero; any occurrence = Tier 1 |
| `DB-CONSTRAINT` | DB unique constraint violation on insert (pre-dedup miss) | T2 | Very low; acceptable if dedup pipeline catches it in Step 2 |
| `RUN-PARTIAL` | Pipeline exited mid-run without completing all 9 steps | T1 | Should be zero |
| `RUN-EMPTY` | Pipeline completed all 9 steps but ingested 0 new games (unexpected) | T1 | Flag if not explained by off-season or holiday calendar |
| `CRON-MISSED` | Cron job did not fire at scheduled time (02:00 weekdays / 23:00 Sunday) | T1 | Should be zero |
| `STEP-DEDUP` | Step 2 dedup merge count anomalously high (> 500 in a single run) | T2 | Very low; investigate for cross-source collision |
| `STEP-ELO` | Glicko-2/ELO computation returned NaN or out-of-range rating value | T1 | Should be zero; indicates data integrity issue upstream |

---

## Section 3: Reporting Protocol

### Standard one-line error summary (every briefing, every run)

```
Error summary: [N] Tier 1, [N] Tier 2, [N] Tier 3 (routine). [Escalation status.]
```

**Examples:**
- `Error summary: 0 Tier 1, 0 Tier 2, 7 Tier 3 (routine). No escalation.`
- `Error summary: 0 Tier 1, 1 Tier 2 (GS-RATELIMIT × 3, org: TYSA), 4 Tier 3. Monitoring — 1st occurrence.`
- `Error summary: 1 Tier 1 (RUN-EMPTY — 0 games, expected off-season). SENTINEL notified.`

---

### Tier 1 format

```
CRITICAL: [code] — [component/source] — [timestamp] — [description of what failed].
Games affected: [N].
SENTINEL notification: YES.
```

**Example:**
```
CRITICAL: RUN-EMPTY — run-daily.sh — 2026-05-03 02:00 — Pipeline completed all 9 steps but returned 0 new games. Not explained by calendar (active spring season).
Games affected: 0.
SENTINEL notification: YES.
```

---

### Tier 2 format

```
[T2] [code] × [count] — [source/component] — last occurrence: [time]. Games impacted: [estimate]. Status: [monitoring / escalating].
```

**Example:**
```
[T2] GS-TIMEOUT × 2 — org: Cal North Soccer — last occurrence: 02:14. Games impacted: ~30–80 (org volume estimate). Status: monitoring, 1st occurrence.
```

---

### Tier 3 format

```
[T3] [code] × [count]. Within expected range: [yes/no].
```

**Example:**
```
[T3] GS-404 × 12, GS-EMPTY × 9, PARSE-SCORE × 1. Within expected range: yes.
```

---

## Section 4: Escalation Thresholds

| Condition | Action |
|-----------|--------|
| Any Tier 1 error | File CRITICAL to SENTINEL in same-session briefing. No GREEN status. |
| Tier 2 error on 2+ consecutive daily runs | Escalate to SENTINEL with error pattern and affected source. |
| Tier 3 count > 25 in a single run | Flag as anomalous in briefing. Do not escalate unless repeats on 2+ consecutive runs. |
| Any source returning 0 games for 2+ consecutive runs | Escalate to SENTINEL regardless of tier — may indicate org deactivation or GotSport platform change. |
| `DLQ-ENTRY` count > 10 in a single run | Flag to SENTINEL — may indicate rate limit policy change. |
| `STEP-ELO` on any run | Tier 1 escalation + immediate halt of that run's downstream outputs. Do not publish rating updates from a run with `STEP-ELO`. |

---

## Section 5: Stage Status Mapping

How error counts map to the GREEN/YELLOW/RED system used in the First-Week Pipeline Monitoring Log and Morning-After Runbook:

| State | Condition |
|-------|-----------|
| ✅ GREEN | 0 Tier 1, 0 Tier 2 (or 1 Tier 2 that is first occurrence), ≤ 25 Tier 3 |
| ⚠️ YELLOW | 0 Tier 1, 1–2 Tier 2 on 2+ consecutive runs, OR Tier 3 > 25 twice in a row |
| 🔴 RED | Any Tier 1 error, OR Tier 2 error persisting 3+ consecutive runs, OR > 0 `STEP-ELO` events |

---

*FORGE — 2026-04-26*
