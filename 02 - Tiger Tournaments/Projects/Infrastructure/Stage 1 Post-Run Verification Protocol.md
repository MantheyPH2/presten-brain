---
title: Stage 1 Post-Run Verification Protocol
tags: [infrastructure, gotsport, stage1, verification, pipeline, evo-draw, forge]
created: 2026-04-27
author: FORGE
status: ready — awaiting Stage 1 execution trigger
---

# Stage 1 Post-Run Verification Protocol

> **Purpose:** Step-by-step protocol FORGE executes within one briefing cycle of Stage 1 (TYSA TX crawl) completing. Produces a formal verification statement that gates SENTINEL's Stage 2 Section 1 authorization assessment.

---

## Section 1: Trigger Conditions

**Trigger:** Presten confirms Stage 1 TYSA crawl ran (via FORGE briefing response or vault file update indicating the crawl executed).

**Target execution window:** Within 30 minutes of trigger receipt.

**Who executes:** FORGE owns the protocol and verification statement. Queries requiring psql access must be run by Presten — FORGE will provide the exact commands.

**Autonomy split:**

| Verification step | Who executes |
|------------------|-------------|
| Review crawl log for error-free completion | FORGE (reads log file or Presten-shared output) |
| Run SQL queries Q1–Q4 | Presten (psql required) |
| Assess results against thresholds | FORGE |
| File verification statement | FORGE |

---

## Section 2: Verification Queries

Run these in sequence immediately after Stage 1 completes. Replace `[TYSA_ORG_ID]` with the confirmed numeric org-ID from the browser session. Replace `[Stage_1_run_date]` with the actual run timestamp.

```sql
-- Query 1: Confirm new games ingested from Stage 1
SELECT COUNT(*) AS new_games, MIN(scraped_at) AS first_scraped, MAX(scraped_at) AS last_scraped
FROM games
WHERE source_org_id = '[TYSA_ORG_ID]'
AND scraped_at > '[Stage_1_run_date]';

-- Query 2: Confirm game date range (should include current season)
SELECT MIN(game_date) AS earliest, MAX(game_date) AS latest
FROM games
WHERE source_org_id = '[TYSA_ORG_ID]'
AND scraped_at > '[Stage_1_run_date]';

-- Query 3: Confirm team count (Texas teams should appear)
SELECT COUNT(DISTINCT team_id) AS teams_seen
FROM games
WHERE source_org_id = '[TYSA_ORG_ID]'
AND scraped_at > '[Stage_1_run_date]';

-- Query 4: Dedup check — confirm no duplicate game records from this crawl
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE source_org_id = '[TYSA_ORG_ID]'
AND scraped_at > '[Stage_1_run_date]'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1;
```

---

## Section 3: Pass/Fail Thresholds

GREEN floor for Stage 1: **≥ 5,000 new games** (sourced from `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` Section 3).

| Check | GREEN (pass) | YELLOW (investigate) | RED (fail) |
|-------|-------------|---------------------|-----------|
| New games ingested (Q1) | ≥ 5,000 | 500–4,999 | < 500 or 0 |
| Game date range (Q2) | Latest game_date in 2025–2026 range | Mixed current and old season | Only pre-2025 dates, or future dates only |
| Teams seen (Q3) | > 50 distinct teams | 10–50 teams | < 10 teams |
| Duplicate rows (Q4) | 0 rows returned | 1–5 rows | > 5 rows |

**GREEN floor rationale:** 5,000 is ~10% of the low-end annual TYSA volume estimate. A healthy crawl hitting even state cups only should clear this threshold. Source: `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md`.

---

## Section 4: FORGE Verification Statement

FORGE fills this template and files it in the active briefing and as a standalone note at `Infrastructure/Stage 1 — FORGE Verification Statement.md`.

```
FORGE Stage 1 Verification Statement
Filed: [date] [time]
Stage 1 run confirmed: [date/time of run]

Verification results:
- New games ingested: [N] — [GREEN / YELLOW / RED]
- Game date range: [earliest date] to [latest date] — [GREEN / YELLOW / RED]
- Teams seen: [N distinct teams] — [GREEN / YELLOW / RED]
- Duplicate rows: [N rows returned from Q4] — [GREEN / YELLOW / RED]

Overall Stage 1 result: [PASS / CONDITIONAL PASS / FAIL]

Notes: [any qualifying remarks on YELLOW results]

FORGE certification: Stage 1 TYSA crawl is confirmed [successful / partially successful / failed].
Stage 2 Section 1 gate: [OPEN for SENTINEL assessment / HOLD — escalating to SENTINEL]

Signed: FORGE — [date]
```

**If any check is RED:** FORGE does not file this statement. Instead, FORGE immediately files a SENTINEL queue item (priority: urgent, category: alert) with the specific failing query output and holds Stage 2 Section 1.

---

## Section 5: Post-Verification Actions

After filing the verification statement, FORGE takes these actions in order:

1. Update `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — confirm TYSA as active in Section 1 with actual game count recorded.
2. Update `Infrastructure/FORGE-Blocked-Task-Dependency-Tracker.md` — Stage 1 completion unblocks Stage 2 Section 1 gate and Stage 2 5.2/5.3 criteria readiness.
3. Open `Infrastructure/May 1 Deployment — Day-Of Runbook.md` and replace the Stage 1 expected count placeholder with: `"Expected new games per run: [Stage 1 actual count] ± 20% tolerance. Source: Stage 1 results [date]."` (per `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` Section 5).
4. File briefing flag: `"Stage 1 PASS — Stage 2 Section 1 gate is OPEN for SENTINEL"` (if PASS) or `"Stage 1 FAIL — escalated to SENTINEL, Stage 2 Section 1 HOLD"` (if FAIL).
5. If PASS: begin working toward Section 5 criteria — 5.2 (crawl test complete — Stage 1 itself qualifies) and 5.3 (Stage 1 regression check now schedulable).

---

## References

- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — volume thresholds for Section 3
- `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` — Section 1 gate conditions this protocol supports
- `Infrastructure/Stage 2 Org-ID Duplicate Cross-Check.md` — Section 5.1 companion verification document
- `Infrastructure/FORGE-Blocked-Task-Dependency-Tracker.md` — dependency chain unlocked by Stage 1 completion
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — receives expected count from Stage 1 actuals

*FORGE — 2026-04-27*
