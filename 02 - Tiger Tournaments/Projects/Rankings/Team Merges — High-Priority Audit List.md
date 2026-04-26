---
title: Team Merges — High-Priority Audit List
type: audit-list
author: ELO
date: 2026-04-26
status: awaiting-query-execution
related: "[[Team Merges — Audit Classification Framework]]", "[[Team Merges — Presten Execution Package]]"
tags: [team-merges, data-quality, audit, evo-draw, rankings]
---

# Team Merges — High-Priority Audit List

> **Status:** Section 1 (filter methodology + SQL) complete. Section 2 (audit list rows) requires Presten to run the three filter queries in psql and paste the results here. Run window: April 29 – May 5, after April 28 recompute confirmed complete.
>
> **Why high-priority:** 27,820 `team_merges` entries cannot be reviewed individually. This document narrows scope to the 30–50 entries most likely to be incorrect — cross-gender merges, large age gaps, and cross-club merges — using the classification framework filed April 26. Clearing these before May 16 (DSS) removes the risk of a merge anomaly appearing in the demo.

---

## Section 1: Filter Methodology

ELO applies the classification framework (`Team Merges — Audit Classification Framework.md`) to generate three tiers of high-priority candidates. All queries assume the post-April-28 recompute has completed — ratings used in the delta calculations reflect the corrected GA ASPIRE calibration.

---

### Tier A — Almost Certainly Wrong

Merges in this tier violate the core invariant of team_merges: merged records should represent the same team. These are executed as three separate queries and their results combined.

**Tier A-1: Cross-gender merges**

Two merged records with different gender designations. A team's gender cannot change — any cross-gender merge is an error.

```sql
SELECT
  tm.id AS merge_id,
  t1.name AS team_a_name,
  t2.name AS team_b_name,
  t1.age_group AS age_group_a,
  t2.age_group AS age_group_b,
  t1.gender AS gender_a,
  t2.gender AS gender_b,
  ABS(COALESCE(r1.rating, 1200) - COALESCE(r2.rating, 1200)) AS rating_delta,
  'A' AS tier,
  'Cross-gender merge' AS risk_reason
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
LEFT JOIN rankings r1 ON r1.team_id = t1.id AND r1.age_group = t1.age_group
LEFT JOIN rankings r2 ON r2.team_id = t2.id AND r2.age_group = t2.age_group
WHERE t1.gender IS NOT NULL
  AND t2.gender IS NOT NULL
  AND t1.gender <> t2.gender
ORDER BY rating_delta DESC;
```

**Tier A-2: Age gap ≥ 2 years**

Merged records where the age group designations differ by 2 or more years (e.g., U14G merged with U16G). Same team at different years does happen in legitimate roster updates, but a 2-year gap is almost always an error.

```sql
SELECT
  tm.id AS merge_id,
  t1.name AS team_a_name,
  t2.name AS team_b_name,
  t1.age_group AS age_group_a,
  t2.age_group AS age_group_b,
  t1.gender AS gender_a,
  t2.gender AS gender_b,
  ABS(COALESCE(r1.rating, 1200) - COALESCE(r2.rating, 1200)) AS rating_delta,
  'A' AS tier,
  'Age gap ≥ 2 years: ' || t1.age_group || ' merged with ' || t2.age_group AS risk_reason
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
LEFT JOIN rankings r1 ON r1.team_id = t1.id AND r1.age_group = t1.age_group
LEFT JOIN rankings r2 ON r2.team_id = t2.id AND r2.age_group = t2.age_group
WHERE t1.age_group IS NOT NULL
  AND t2.age_group IS NOT NULL
  AND t1.age_group <> t2.age_group
  AND ABS(
    CAST(REGEXP_REPLACE(t1.age_group, '[^0-9]', '', 'g') AS INTEGER) -
    CAST(REGEXP_REPLACE(t2.age_group, '[^0-9]', '', 'g') AS INTEGER)
  ) >= 2
ORDER BY rating_delta DESC;
```

**Tier A-3: Cross-club merges**

Two records from different `club_id` values merged together. This combines rating histories from different clubs — almost always an error unless the club itself merged (which would be reflected in `club_merges`, not `team_merges`).

```sql
SELECT
  tm.id AS merge_id,
  t1.name AS team_a_name,
  t2.name AS team_b_name,
  t1.age_group AS age_group_a,
  t2.age_group AS age_group_b,
  t1.gender AS gender_a,
  t2.gender AS gender_b,
  ABS(COALESCE(r1.rating, 1200) - COALESCE(r2.rating, 1200)) AS rating_delta,
  'A' AS tier,
  'Cross-club: club_id ' || t1.club_id::text || ' merged with ' || t2.club_id::text AS risk_reason
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
LEFT JOIN rankings r1 ON r1.team_id = t1.id AND r1.age_group = t1.age_group
LEFT JOIN rankings r2 ON r2.team_id = t2.id AND r2.age_group = t2.age_group
WHERE t1.club_id IS NOT NULL
  AND t2.club_id IS NOT NULL
  AND t1.club_id <> t2.club_id
ORDER BY rating_delta DESC
LIMIT 50;
```

*Note: Cap Tier A-3 at 50 rows (highest rating delta first). Report the total cross-club merge count in the SENTINEL briefing summary.*

---

### Tier B — Probably Wrong or Needs Verification

**Tier B: 1-year age gap OR large rating delta (> 150 pts) with same club**

```sql
SELECT
  tm.id AS merge_id,
  t1.name AS team_a_name,
  t2.name AS team_b_name,
  t1.age_group AS age_group_a,
  t2.age_group AS age_group_b,
  t1.gender AS gender_a,
  t2.gender AS gender_b,
  ABS(COALESCE(r1.rating, 1200) - COALESCE(r2.rating, 1200)) AS rating_delta,
  'B' AS tier,
  CASE
    WHEN t1.age_group <> t2.age_group THEN 'Age gap 1 year: ' || t1.age_group || ' / ' || t2.age_group
    ELSE 'Rating delta > 150 pts same-club merge'
  END AS risk_reason
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
LEFT JOIN rankings r1 ON r1.team_id = t1.id AND r1.age_group = t1.age_group
LEFT JOIN rankings r2 ON r2.team_id = t2.id AND r2.age_group = t2.age_group
WHERE t1.club_id = t2.club_id -- same club (exclude Tier A-3)
  AND t1.gender = t2.gender   -- same gender (exclude Tier A-1)
  AND (
    (t1.age_group <> t2.age_group
      AND ABS(
        CAST(REGEXP_REPLACE(t1.age_group, '[^0-9]', '', 'g') AS INTEGER) -
        CAST(REGEXP_REPLACE(t2.age_group, '[^0-9]', '', 'g') AS INTEGER)
      ) = 1)
    OR
    ABS(COALESCE(r1.rating, 1200) - COALESCE(r2.rating, 1200)) > 150
  )
ORDER BY tier, rating_delta DESC
LIMIT 30;
```

---

### Tier C — Worth Reviewing

**Tier C: Same club, same age/gender, but rating delta 100–150 pts**

*Low priority. Include only if Tiers A + B produce < 30 rows total.*

```sql
SELECT
  tm.id AS merge_id,
  t1.name AS team_a_name,
  t2.name AS team_b_name,
  t1.age_group AS age_group_a,
  t2.age_group AS age_group_b,
  t1.gender AS gender_a,
  t2.gender AS gender_b,
  ABS(COALESCE(r1.rating, 1200) - COALESCE(r2.rating, 1200)) AS rating_delta,
  'C' AS tier,
  'Same club / age / gender but rating delta 100–150 pts' AS risk_reason
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
LEFT JOIN rankings r1 ON r1.team_id = t1.id AND r1.age_group = t1.age_group
LEFT JOIN rankings r2 ON r2.team_id = t2.id AND r2.age_group = t2.age_group
WHERE t1.club_id = t2.club_id
  AND t1.gender = t2.gender
  AND t1.age_group = t2.age_group
  AND ABS(COALESCE(r1.rating, 1200) - COALESCE(r2.rating, 1200)) BETWEEN 100 AND 150
ORDER BY rating_delta DESC
LIMIT 20;
```

---

## Section 2: High-Priority Audit List

> **STATUS: AWAITING PRESTEN QUERY EXECUTION.** Run the three Tier A queries and the Tier B query in psql after April 28 recompute is confirmed complete (window: April 29 – May 5). Paste output rows into this table. Save raw output to `02 - Tiger Tournaments/Projects/Reports/team-merges-tier-a-[date].md` and `team-merges-tier-b-[date].md`.

| Merge ID | Team A Name | Team B Name | Age Group A | Age Group B | Gender A | Gender B | Rating Delta | Tier | Risk Reason |
|----------|------------|------------|-------------|-------------|----------|----------|-------------|------|-------------|
| *— awaiting Presten psql run —* | | | | | | | | | |

**Target rows:** 30–50 total. If Tier A alone returns > 50 rows, cap at 50 (highest rating delta first) and report total Tier A count in ELO briefing.

**After receiving Presten's output:** ELO applies the classification framework decision tree to each row and adds `Type` and `Severity` columns. File updated table in the next ELO briefing.

---

## Section 3: Presten Review Package

Once ELO classifies all rows and returns the table to Presten:

**Standard review question for each row:** "Are [Team A Name] and [Team B Name] the same team?"

**For Tier A entries:** "These teams have [cross-gender / cross-club / large age gap]. Unless you have specific knowledge of an unusual situation (e.g., a known club rebrand that crossed age groups), this merge should be **REVERSED**."

**For Tier B entries:** "These teams are from the same club but [age groups differ by 1 year / rating spread is unusually large]. Please verify: are these the same team roster under different names, or distinct teams?"

**For Tier C entries:** "Low priority — review only if time allows."

**Presten marks each row with one of:**
- `CONFIRM` — merge is correct, rating history contamination is acceptable
- `REVERSE` — merge is incorrect, should be split into two separate teams
- `INVESTIGATE` — unclear; ELO files a Queue item with the specific ambiguity

**REVERSE decisions:** ELO runs the unmerge SQL from `Team Merges — Presten Execution Package.md` for each flagged row and triggers a recompute for the affected teams. These should be completed before May 9 DSS readiness check.

---

## Section 4: Expected Impact Assessment

*(ELO fills this section after receiving Tier A query results — based on classification, not live database access. Estimates are derived from the classification framework and calibration context.)*

**Reversing Tier A-1 (cross-gender merges):**
- Impact direction: Mixed. Cross-gender merges contaminate the affected team's rating with games from a differently-calibrated gender pool. The direction of impact depends on which gender's history dominates the merged record. Expect rating corrections of 20–60 points per affected team.
- Age groups most affected: Likely concentrated in younger age groups (U12–U14) where gender-neutral name conventions (e.g., "FC Dallas Select U13") may have caused the matching error.

**Reversing Tier A-3 (cross-club merges):**
- Impact direction: Likely upward correction for the lower-rated team in the merged pair (their inflated rating, borrowed from the other club's history, drops back to a clean baseline). Estimated 40–100 points per affected team.
- Most impactful: Merges involving teams with large rating deltas (> 200 pts) between the two merged records.

**Demo team risk:**
- Presten should flag if Football Academy NJ, PA Classics, PDA, NJ Wildcats, or FC Stars appear in any row of the audit list (any tier). These are DSS demo anchor teams — any merge anomaly involving them must be corrected before May 14.
- ELO will cross-check demo team names against the audit list immediately upon receiving results.

*(Update this section with specific numbers once Presten returns query output.)*

---

## Section 5: Audit Status Tracker

| Tier | Total Entries | Confirmed Correct | Reversed | Under Investigation | Not Yet Reviewed |
|------|--------------|------------------|---------|---------------------|-----------------|
| A | *fill after Presten runs queries* | | | | |
| B | | | | | |
| C | | | | | |
| **Total** | | | | | |

ELO updates this table in each briefing until "Not Yet Reviewed" reaches 0 for Tier A. SENTINEL monitors Tier A completion as the DSS data quality gate.

**Target:** All Tier A rows reviewed and actioned before May 9 DSS readiness check. Tier B reviews before May 14 (pre-demo validation day).

---

## References

- [[Team Merges — Audit Classification Framework]] — type taxonomy and decision tree ELO applies to each row
- [[Team Merges — Presten Execution Package]] — unmerge SQL and query execution context
- `task-2026-04-23-team-merges-verification-campaign.md` — original campaign that established the 27,820 count
- [[Rank Bands Demo Examples]] — demo team names ELO cross-checks against audit rows
- [[Club Rankings — Reference Clubs]] — additional demo clubs to include in cross-check

*ELO — 2026-04-26*
