---
type: elo-checkpoint-results
topic: ecnl-cp1
date-scheduled: 2026-04-30
date-executed: "[TO FILL: date CP1 runs]"
status: pending-execution
option-recommendation: Option 1 (No Re-tag)
task: task-2026-04-29-ecnl-cp1-result-filing-protocol
related-brief: "Rankings/ECNL Migration Option — Decision Brief.md"
tags: [elo, ecnl, migration, checkpoint, cp1, option-decision, evo-draw]
---

# ECNL CP1 — Checkpoint Results

> **Context:** CP1 slipped from April 29 to April 30 and is the first action of that session. CP1 tests whether Option 1 (No Re-tag) is viable from an ECNL data continuity standpoint. Result may confirm ELO's Option 1 recommendation or introduce evidence shifting toward Option 2. ELO files this document within 30 minutes of CP1 completion.
>
> **Decision deadline:** April 30 EOD — Presten must authorize Option 1 or Option 2 for ECNL migration.

---

## Section 1 — CP1 Test Results

ELO pre-filled Expected column from `Rankings/ECNL Rating Continuity Spec — June 2026.md` and `Rankings/ECNL Migration — Option Comparison Matrix.md` (CP2 criteria).

| Check | Expected | Actual | Pass/Fail |
|-------|----------|--------|-----------|
| ecnl_verified backfill row count | > 0 rows; matches Path A team count from April 29 Execution Log Step 2b. Disqualifying threshold: < 70% of expected Path A team count. | (fill on April 30) | (fill) |
| Teams correctly marked ecnl_verified = TRUE | All ECNL teams active this season (Path A teams migrating TGS → GotSport) have ecnl_verified = TRUE. No active ECNL team should be unmarked. | (fill on April 30) | (fill) |
| No ECNL teams incorrectly excluded (false negatives) | Zero false negatives. Every team in the current ECNL roster with an active game record this season carries ecnl_verified = TRUE. Source: Rating Continuity Spec — Phase 1 loads verified merge records only; a missing flag = invisible team history. | (fill on April 30) | (fill) |
| Rating continuity: sample team rating pre/post | No rating change for any ECNL sample team. Option 1 = No Re-tag = no recompute of historical games. Pre-CP1 rating must equal post-CP1 rating for a sample of 3–5 known ECNL teams. (Compare to pre-April-28 baseline ratings from `Rankings/Pre-April-28 Baseline Snapshot Spec.md` if available.) | (fill on April 30) | (fill) |

**Confirmation queries (run April 30):**

```sql
-- Check 1 & 2: ecnl_verified backfill count and coverage
SELECT 
  ecnl_verified,
  COUNT(*) AS team_count
FROM teams
WHERE ecnl_verified IS NOT NULL
GROUP BY ecnl_verified;
-- Expected: one row with ecnl_verified = TRUE, count > 0

-- Alternative if ecnl_verified is on games table:
SELECT ecnl_verified, COUNT(*) 
FROM games 
WHERE ecnl_verified IS NOT NULL 
GROUP BY ecnl_verified;
```

```sql
-- Check 3: False negative detection (ECNL teams without flag)
-- Adapt ECNL league/source filter to match actual schema
SELECT t.name, r.age_group, r.gender, r.rating, r.rank
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE t.source = 'ecnl'                     -- or equivalent source/league column
  AND (t.ecnl_verified IS NULL OR t.ecnl_verified = FALSE)
  AND r.rank IS NOT NULL
ORDER BY r.age_group, r.rank
LIMIT 20;
-- Expected: 0 rows
```

```sql
-- Check 4: Rating continuity for 3–5 known ECNL teams
-- Replace team names with actual known ECNL teams
SELECT t.name, r.age_group, r.gender, r.rating, r.rank, r.last_game_date
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE t.name ILIKE ANY (ARRAY[
  '%Solar SC%',
  '%FC Dallas%',
  '%Real Colorado%',
  '%San Juan ECNL%',
  '%Pipeline%'
])
  AND r.gender = 'F'
ORDER BY r.age_group, r.rank;
-- Compare to pre-April-28 snapshot values
-- Expected: ratings unchanged from pre-session baseline
```

---

## Section 2 — CP1 Verdict

*(Fill within 30 minutes of CP1 execution. Apply the relevant path below.)*

**CP1 PASS:** All 4 checks pass → Option 1 remains the correct recommendation. No change to `Rankings/ECNL Migration Option — Decision Brief.md`. Notify SENTINEL: "CP1 PASS — Option 1 recommendation confirmed. Presten can authorize Option 1 by April 30 EOD."

**CP1 CONDITIONAL:** One check fails non-critically (e.g., backfill count differs from expected by < 5%, or 1–2 sample teams show minor rating delta ≤ 10 pts) → ELO notes the deviation, states whether it affects Option 1 viability, and recommends whether Presten should still authorize Option 1 or wait for CP2/CP3 evidence. The decision brief is updated with a CP1 Addendum.

**CP1 FAIL:** A critical check fails (e.g., backfill < 70% complete, > 2 sample teams show rating change, material false negatives detected) → ELO immediately: (1) adds a STOP notice at the top of `Rankings/ECNL Migration Option — Decision Brief.md`, (2) notifies SENTINEL via Queue, (3) files this document with FAIL verdict and a recommended pivot analysis toward Option 2. April 30 authorization decision may need to defer until CP2 (ecnl_verified accuracy spot check, May 5) provides more evidence.

**Actual CP1 Verdict:** [TO FILL: PASS / CONDITIONAL / FAIL — April 30]

---

## Section 3 — Decision Brief Update (if needed)

*(Complete within 30 minutes of CP1 result.)*

**If CP1 PASS:** No update to Decision Brief required. Note here: "Decision Brief unchanged. CP1 PASS on [date]."

**If CP1 CONDITIONAL or FAIL:** Edit `Rankings/ECNL Migration Option — Decision Brief.md` as follows:
- Append CP1 Results Addendum at the bottom of the brief
- Update Options at a Glance table: Option 1 — Risk column updated to reflect CP1 finding
- Add a flag at the top of the brief if CP1 FAIL: "> ⚠️ CP1 FAIL — See CP1 Results Addendum before authorizing."

Decision Brief update status: [TO FILL: No update required / Updated — see Decision Brief CP1 Addendum]

---

## Section 4 — ELO Recommendation Post-CP1

*(Fill within 30 minutes of CP1 result.)*

[TO FILL: One sentence confirming or revising the Option 1 recommendation. Do not hedge — state the recommendation clearly.]

*Pre-draft (complete on CP1 PASS — most likely path):*
CP1 PASS confirms that the ecnl_verified backfill is complete and Option 1's core data dependency is met. ELO recommendation is unchanged: **proceed with Option 1 (No Re-tag)**. Presten should authorize Option 1 by April 30 EOD per the Decision Brief.

*Pre-draft (complete on CP1 FAIL — pivot path):*
CP1 FAIL indicates the ecnl_verified backfill has a critical gap and Option 1's Path A identification mechanism is unreliable. ELO recommendation shifts to **HOLD pending CP5 evidence** (ecnl_verified accuracy spot check, May 5). Presten should not authorize Option 1 until FORGE confirms a fix timeline. Option 2 remains viable if ecnl_transition records in team_merges are complete.

---

## Section 5 — SENTINEL Notification

*(Send in the next ELO briefing after CP1 runs.)*

**Template (fill on April 30):**
> "CP1 result filed [date]. Recommendation: [Option 1 / Option 2 / HOLD pending CP2]. Presten authorization can [proceed / should pause pending [reason]]. ECNL Decision Brief [unchanged / updated — see Addendum]. Next checkpoint: CP3 (first pipeline ECNL ingestion, May 1)."

Actual notification: [TO FILL]

---

## Filing Sequence — April 30

1. April 30 session opens → CP1 check runs (FORGE confirms ecnl_verified backfill per April 29 Execution Log Step 2b, if not already confirmed)
2. FORGE or Presten confirms ecnl_verified column state and backfill count
3. ELO runs rating continuity check (Check 4 query above) and false negative query (Check 3)
4. ELO fills Sections 1–5 of this document (target: < 30 minutes from data receipt)
5. ELO updates Decision Brief if needed
6. ELO notifies SENTINEL in next briefing

**If April 30 session does not open:**
ELO files a Queue item to SENTINEL noting CP1 slip to next session and impact on April 30 EOD authorization deadline. Option 1 vs. 2 decision deferred; notify Presten that April 30 EOD deadline cannot be met without CP1 data.

---

## References

- `Rankings/ECNL Migration Option — Decision Brief.md` — authorization gate document; updated if CP1 CONDITIONAL or FAIL
- `Rankings/ECNL Migration — Option Comparison Matrix.md` — full 7-checkpoint campaign; this document covers CP1/CP2
- `Rankings/ECNL Rating Continuity Spec — June 2026.md` — expected values source for Checks 1–4
- `Rankings/ECNL Migration — CP1 Fail Escalation Protocol.md` — escalation procedures if CP1 FAIL
- `task-2026-04-28-ecnl-cp1-cp2-checkpoint-results-document.md` — parent tracking task

*ELO — 2026-04-29 (pre-draft; Expected column pre-filled)*
