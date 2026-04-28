---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 Post-Run Verification Protocol.md"
---

# Task: Stage 1 Post-Run Verification Protocol

## Context

FORGE has filed the Stage 1 TX Crawl Expected Output Spec — it defines what a successful Stage 1 run should produce. What does not exist: a post-run verification protocol that FORGE executes immediately after Stage 1 runs to confirm it actually succeeded.

The Stage 2 authorization process requires Stage 1 to be confirmed complete and successful before SENTINEL can assess Section 1 of the authorization criteria. That confirmation must come from FORGE in the form of a verification statement. Right now, if Stage 1 ran today, FORGE would have to improvise the verification steps. That improvisation introduces risk: missing a query, misreading a result, or filing a verification statement that doesn't address what SENTINEL actually needs to see.

This document closes that gap. It gives FORGE a step-by-step protocol to execute within one briefing cycle of Stage 1 completing.

---

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 1 Post-Run Verification Protocol.md`

---

### Section 1: Trigger Conditions

Define exactly when this protocol activates:

- Trigger: Presten confirms Stage 1 TYSA crawl ran (in FORGE's briefing response or via vault file update)
- Target execution window: within 30 minutes of trigger
- Who runs it: FORGE (documentation only — queries may require Presten to execute in psql)

Identify which queries FORGE can verify autonomously (by reading pipeline logs or vault files) versus which require Presten to run against the database.

---

### Section 2: Verification Queries

Provide the specific SQL queries FORGE needs Presten to run, in execution order:

```sql
-- Query 1: Confirm new games ingested from Stage 1
SELECT COUNT(*) AS new_games, MIN(scraped_at) AS first_scraped, MAX(scraped_at) AS last_scraped
FROM games
WHERE org_id = [TYSA_ORG_ID]
AND scraped_at > [Stage_1_run_date];

-- Query 2: Confirm game date range (should be current season)
SELECT MIN(game_date) AS earliest, MAX(game_date) AS latest
FROM games
WHERE org_id = [TYSA_ORG_ID]
AND scraped_at > [Stage_1_run_date];

-- Query 3: Confirm team count (Texas teams should appear)
SELECT COUNT(DISTINCT team_id) AS teams_seen
FROM games
WHERE org_id = [TYSA_ORG_ID]
AND scraped_at > [Stage_1_run_date];

-- Query 4: Confirm no duplicate key errors (check for existing games at same org_id)
SELECT COUNT(*) AS pre_existing_games
FROM games
WHERE org_id = [TYSA_ORG_ID]
AND scraped_at < [Stage_1_run_date];
```

Replace bracket placeholders with actual values once confirmed. Add any additional queries FORGE identifies as necessary.

---

### Section 3: Pass/Fail Thresholds

Define explicit thresholds for each query. Cross-reference the Expected Output Spec for the volume numbers.

| Check | GREEN (pass) | YELLOW (investigate) | RED (fail) |
|-------|-------------|---------------------|-----------|
| New games ingested | ≥ GREEN floor from Output Spec | 10–49% of expected | < 10% of expected or 0 |
| Game date range | Current season dates present | Mixed season/off-season | Only off-season or future |
| Teams seen | > 50 distinct teams | 10–50 teams | < 10 teams |
| Pre-existing games | 0 (first crawl) | 1–100 (minor overlap) | > 100 (unexpected overlap) |

FORGE must translate the Output Spec GREEN floor into the actual number here. Do not leave it as "≥ GREEN floor."

---

### Section 4: FORGE Verification Statement

Template for the statement FORGE files in the FORGE briefing and in the Stage 2 Authorization Criteria document (Section 6 support):

```
FORGE Stage 1 Verification Statement
Filed: [date] [time]
Stage 1 run confirmed: [date/time of run]

Verification results:
- New games ingested: [N] — [GREEN / YELLOW / RED]
- Game date range: [date range] — [GREEN / YELLOW / RED]
- Teams seen: [N] — [GREEN / YELLOW / RED]
- Pre-existing games: [N] — [GREEN / YELLOW / RED]

Overall Stage 1 result: [PASS / CONDITIONAL PASS / FAIL]

FORGE certification: Stage 1 TYSA crawl is confirmed [successful / partially successful / failed].
Stage 2 Section 1 gate: [OPEN for SENTINEL assessment / HOLD — escalating to SENTINEL]

Signed: FORGE — [date]
```

If any check is RED, FORGE does not file this statement — it files a queue item to SENTINEL immediately and holds Stage 2 Section 1.

---

### Section 5: Post-Verification Actions

After filing the verification statement, FORGE takes these actions in order:

1. Mark `task-2026-04-27-stage1-post-run-verification-protocol.md` status: completed
2. Update `Infrastructure/FORGE-Blocked-Task-Dependency-Tracker.md` — Stage 1 completion unblocks dependency item #[N]
3. File a briefing flag: "Stage 1 PASS — Stage 2 Section 1 gate is open for SENTINEL"
4. If Stage 1 PASS: FORGE begins working toward Section 5 criteria verification (5.2 crawl test, 5.3 Stage 1 regression check are now completable)
5. If Stage 1 FAIL: FORGE escalates to SENTINEL via queue item before any next session action

---

## Quality Standard

- All thresholds must be specific numbers, not ranges so wide they'd pass any result.
- The verification statement template must be fillable by FORGE without additional research — all context for what to fill in must be in this document or the Output Spec.
- The protocol must be executable within 30 minutes of Stage 1 running. If it requires more than 30 minutes, it is too complex.

## Why This Matters

SENTINEL cannot assess Stage 2 Section 1 without FORGE's verification that Stage 1 succeeded. If Stage 1 runs at 2am and FORGE has no protocol, the Stage 2 gate assessment is delayed until the next session. With this protocol, FORGE files the verification statement in the same briefing cycle as Stage 1 completion, and SENTINEL can assess Section 1 immediately.

## References

- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — volume thresholds for Section 3
- `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` — Section 1 gate conditions
- `Infrastructure/Stage 2 Org-ID Duplicate Cross-Check.md` — Section 5.1 companion document
- `Infrastructure/FORGE-Blocked-Task-Dependency-Tracker.md` — dependency chain
