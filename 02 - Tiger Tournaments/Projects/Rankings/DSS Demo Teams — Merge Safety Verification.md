---
title: DSS Demo Teams — Merge Safety Verification
type: safety-check
author: ELO
date: 2026-04-26
status: awaiting-query-execution
related: "[[Team Merges — High-Priority Audit List]]", "[[Team Merges — Presten Execution Package]]", "[[Rank Bands — Demo Examples Document]]", "[[Club Rankings Reference Spot-Check Set]]"
tags: [dss, demo, team-merges, data-quality, safety-check, evo-draw]
---

# DSS Demo Teams — Merge Safety Verification

> **Purpose:** Pre-DSS merge safety check for the five named demo teams (Football Academy NJ, PA Classics, PDA, NJ Wildcats, FC Stars of Massachusetts). Run the three queries in Section 2 during any psql session April 29–May 7. Fill Section 3 when results arrive.
>
> **Why this exists:** Section 4 of `Rankings/Team Merges — High-Priority Audit List.md` flagged that demo teams should be cross-checked against the high-risk merge query output before the demo. This document executes that cross-check as a standalone 30-minute task.

---

## Section 1: Demo Teams Under Review

| Team Name | Age Group(s) | Gender | Primary Demo Feature | Known League |
|-----------|-------------|--------|---------------------|-------------|
| Football Academy NJ | U13G (primary); possibly U14G–U16G | F | Rank Bands — primary named example in demo script | GA, ECNL RL |
| PA Classics | All Girls age groups (U13G–U17G) | F | Club Rankings — must appear ≤ rank 30 Girls nationally | ECNL Girls |
| PDA (Premier Development Academy) | All Girls + some Boys age groups | F/M | Club Rankings — NJ flagship, DSS audience geography anchor | ECNL Girls + Boys |
| NJ Wildcats (or NJ Rush) | All Girls age groups | F | Club Rankings — second NJ demo club | ECNL Girls / ECNL RL |
| FC Stars of Massachusetts | All Girls + Boys age groups | F/M | Club Rankings — NE region representative | ECNL Girls + Boys |

> **Note on "NJ Wildcats":** Use whichever variant (`NJ Wildcats` or `NJ Rush`) has more teams in the DB. Query A uses `ILIKE '%NJ Wildcats%'` — run a separate check with `'%NJ Rush%'` if the primary returns 0 results.

---

## Section 2: Merge Check Queries

Run all three queries during the April 29 session or the first available psql session. They operate on the `team_merges` table and require the post-April-28 recompute to be complete.

---

### Query A — Demo Teams as Primary (Canonical) Team in a Merge

```sql
SELECT
  tm.id AS merge_id,
  t1.name AS primary_team_name,
  t1.age_group AS primary_age_group,
  t1.gender AS primary_gender,
  t1.id AS primary_team_id,
  t2.name AS merged_team_name,
  t2.age_group AS merged_age_group,
  t2.gender AS merged_gender,
  t2.id AS merged_team_id,
  tm.method AS merge_method,
  tm.created_at AS merge_date
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
WHERE t1.name ILIKE ANY (ARRAY[
  '%Football Academy NJ%',
  '%Football Academy%NJ%',
  '%PA Classics%',
  '%PDA%',
  '%Premier Development Academy%',
  '%NJ Wildcats%',
  '%NJ Rush%',
  '%FC Stars%'
])
ORDER BY t1.name, tm.created_at;
```

**Expected behavior:** Returns 0 rows if no demo teams are the primary (canonical) team in any merge. Any rows returned must be classified in Query C.

---

### Query B — Demo Teams as Secondary (Absorbed) Team in a Merge

```sql
SELECT
  tm.id AS merge_id,
  t1.name AS primary_team_name,
  t1.age_group AS primary_age_group,
  t1.gender AS primary_gender,
  t1.id AS primary_team_id,
  t2.name AS absorbed_team_name,
  t2.age_group AS absorbed_age_group,
  t2.gender AS absorbed_gender,
  t2.id AS absorbed_team_id,
  tm.method AS merge_method,
  tm.created_at AS merge_date
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
WHERE t2.name ILIKE ANY (ARRAY[
  '%Football Academy NJ%',
  '%Football Academy%NJ%',
  '%PA Classics%',
  '%PDA%',
  '%Premier Development Academy%',
  '%NJ Wildcats%',
  '%NJ Rush%',
  '%FC Stars%'
])
ORDER BY t2.name, tm.created_at;
```

**Important:** Teams returned here have been absorbed into another team. They may NOT appear under their own name in current rankings — the canonical team name is what appears. If PA Classics is absorbed, for example, it may not show in club rankings under "PA Classics." This is a critical demo risk — the demo names specific teams by name.

---

### Query C — Classify Each Merge Found in A or B

For every merge returned by Query A or Query B, run the following classification check using the `merge_id` values returned:

```sql
SELECT
  tm.id AS merge_id,
  t1.name AS primary_name,
  t1.gender AS primary_gender,
  t1.age_group AS primary_age_group,
  t2.name AS secondary_name,
  t2.gender AS secondary_gender,
  t2.age_group AS secondary_age_group,
  CASE
    WHEN t1.gender = t2.gender THEN 'SAME_GENDER'
    ELSE 'CROSS_GENDER'
  END AS gender_classification,
  CASE
    WHEN t1.club_id = t2.club_id THEN 'SAME_CLUB'
    ELSE 'CROSS_CLUB'
  END AS club_classification,
  ABS(
    CAST(SUBSTRING(t1.age_group FROM '[0-9]+') AS INT) -
    CAST(SUBSTRING(t2.age_group FROM '[0-9]+') AS INT)
  ) AS age_gap_years,
  CASE
    WHEN ABS(
      CAST(SUBSTRING(t1.age_group FROM '[0-9]+') AS INT) -
      CAST(SUBSTRING(t2.age_group FROM '[0-9]+') AS INT)
    ) <= 1 THEN 'ADJACENT_AGE'
    ELSE 'AGE_GAP_RISK'
  END AS age_classification,
  CASE
    WHEN t1.gender = t2.gender
      AND t1.club_id = t2.club_id
      AND ABS(
        CAST(SUBSTRING(t1.age_group FROM '[0-9]+') AS INT) -
        CAST(SUBSTRING(t2.age_group FROM '[0-9]+') AS INT)
      ) <= 1
    THEN 'SAFE'
    ELSE 'SUSPICIOUS'
  END AS merge_verdict,
  COALESCE(r1.rating, 0) AS primary_rating,
  COALESCE(r2.rating, 0) AS secondary_rating,
  ABS(COALESCE(r1.rating, 0) - COALESCE(r2.rating, 0)) AS rating_delta
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
LEFT JOIN rankings r1 ON r1.team_id = t1.id AND r1.age_group = t1.age_group
LEFT JOIN rankings r2 ON r2.team_id = t2.id AND r2.age_group = t2.age_group
WHERE tm.id IN ([paste merge_ids from Query A and Query B results here])
ORDER BY merge_verdict DESC, rating_delta DESC;
```

**Safe:** Same gender + same club + age gap ≤ 1 year. This is a legitimate team history merge.
**Suspicious:** Different gender OR different club OR age gap ≥ 2 years. This may distort rankings.

---

## Section 3: Verification Results

> Fill when Presten runs Queries A, B, C. Leave blank at document creation.

| Team Name | Found as Primary? | Found as Secondary? | Merge Count | Worst Merge Classification | SAFE / SUSPICIOUS |
|-----------|------------------|--------------------|-----------|-----------------------------|-------------------|
| Football Academy NJ | | | | | |
| PA Classics | | | | | |
| PDA | | | | | |
| NJ Wildcats / NJ Rush | | | | | |
| FC Stars of Massachusetts | | | | | |

**File results note in briefing:** "Demo team merge check complete — results in `Rankings/DSS Demo Teams — Merge Safety Verification.md` Section 3."

---

## Section 4: Risk Assessment

| Result | Implication | Action |
|--------|-------------|--------|
| No demo team appears in any merge | All clear. Demo teams have clean histories. | None. Proceed with demo as planned. |
| Demo team in SAFE merge(s) only | Low risk. Normal same-club age-proximity merge. | Note in SENTINEL briefing: "Demo team [name] has [N] safe merges — no action needed." No demo change. |
| Demo team in SUSPICIOUS merge | Rating may be distorted. Demo team may be ranking higher or lower than their actual performance warrants. | SENTINEL flags to Presten. Assess whether the distortion is large enough to affect the demo (rating delta > 75 pts is the concern threshold). If so, switch to the alternate demo team from Section 5. |
| Demo team absorbed (secondary) and no longer ranking under its known name | The team may appear in rankings under the canonical team name instead of the demo team name. The demo may break — Presten would query "PA Classics" and find it absent. | SENTINEL flags urgently. Either use the canonical team name in the demo, or switch to alternate demo team. Check which name appears in `rankings` for the affected team. |

**Concern threshold for suspicious merges:** Rating delta > 75 pts between the merged teams is a significant distortion risk. Merges with rating_delta < 40 pts are unlikely to visibly affect rankings in the demo.

---

## Section 5: Alternate Demo Teams

Pre-selected backup teams. These do not need merge verification — they are held in reserve in case a suspicious merge disqualifies the primary.

| Primary | Demo Feature | Backup 1 | Backup 2 |
|---------|-------------|----------|----------|
| Football Academy NJ | Rank Bands example (U13G) | **NJ Stallions** (NJ, GA, U13G) — similar profile as a NJ Girls GA program; likely Gold or Silver band | **FC Copa Academy** (NJ, ECNL RL Girls) — strong NJ Girls program with multi-age-group history |
| PA Classics | Club Rankings (Girls, PA) | **Real Colorado** (CO, ECNL Girls) — top 20 nationally, deep multi-age-group program; good geographic contrast | **Michigan Hawks** (MI, ECNL Girls) — top 35 nationally, Midwest anchor; demonstrates rankings work beyond NE |
| PDA | Club Rankings (NJ anchor) | **Sporting Blue Valley** (KS/MO, ECNL Girls) — top 50 nationally; demonstrates the rankings aren't NJ-centric | **FC Dallas Academy** (TX, ECNL Girls) — top 5 nationally; stronger national-tier example if PDA is disqualified |
| NJ Wildcats / NJ Rush | Club Rankings (second NJ club) | **Match Fit Academy** (NJ, ECNL RL Girls) — NJ program with strong ECNL RL presence | **Oakwood SC** (NY, ECNL Girls) — NY tristate area; DSS audience is familiar with NY programs |
| FC Stars of Massachusetts | Club Rankings (NE region) | **Seacoast United** (NH/ME, ECNL RL Girls) — NE regional program, well-known in NE market | **Connecticut FC** (CT, ECNL Girls) — tristate area coverage; familiar to DSS NE audience |

> **Backup selection criteria:** Each backup serves the same demo purpose as the primary (same geography/league tier narrative where possible), is unlikely to have suspicious merge issues (mid-to-top-tier clubs with stable histories), and has enough games for stable rankings (confirmed in the Club Rankings Reference Spot-Check Set or expected from league affiliation).

---

## References

- `Rankings/Team Merges — High-Priority Audit List.md` — Tier A queries and Section 4 demo team note
- `Rankings/Team Merges — Presten Execution Package.md` — psql session context; run these queries in the same session as the Tier A audit
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — demo feature map
- `Rankings/Rank Bands — Demo Examples Document.md` — Football Academy NJ context (primary Rank Bands example)
- `Rankings/Club Rankings Reference Spot-Check Set.md` — PA Classics, PDA context; pass thresholds

*ELO — 2026-04-26*
