---
title: Pre-DSS Team Merge Audit — U13G
aliases: [U13G Merge Audit, DSS Merge Audit]
tags: [rankings, dedup, merges, u13g, dss, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: sql-ready — pending Presten psql session
due: 2026-04-27
---

# Pre-DSS Team Merge Audit — U13G

> **Purpose:** Resolve team identity splits in U13G before Rank Bands implementation. Football Academy NJ / FA 12G Black is the primary case. All 16 USA Rank reference teams are cross-checked for fragmentation. Every SQL in this document is copy-paste ready and sequenced for a single 30–45 minute psql session.

---

## Why This Must Be Done First

USA Rank lists **The Football Academy NJ 2012 Girls Black** at #4 and **FA 12G Black** at #38 in U13G. These are almost certainly the same organization under two different name conventions — "Football Academy NJ" as the full club name and "FA 12G" as an abbreviated form. If Evo Draw has not merged them:

- Neither record accumulates the full game history it should
- The actual #1 or #2 U13G team in the NJ/PA/NE metro may appear fragmented at #60 and #180
- When Rank Bands are applied to this data, a Gold-tier club known to DSS attendees appears as Bronze or lower
- This is exactly the audience at DSS — NJ, PA, NE corridor coaches and club directors

**Resolution path:** Run every query in this document top-to-bottom in a single psql session. No design decisions required — each query has a named action and the merge SQL is pre-written.

---

## Section 1: Football Academy NJ — Full Investigation

### Query 1A: Name Search — Both Variants

```sql
-- Find all teams matching either Football Academy NJ or FA 12G patterns
SELECT
  t.team_id,
  t.team_name,
  r.rating,
  r.rank,
  r.match_count,
  r.last_game_date,
  r.age_group,
  r.gender
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id
WHERE
  t.team_name ILIKE '%Football Academy%'
  OR t.team_name ILIKE '%FA 12G%'
  OR t.team_name ILIKE '%FA12G%'
ORDER BY r.rating DESC NULLS LAST;
```

**Expected outcomes:**
- **Outcome A:** One record found (Football Academy NJ) — already merged or never split. No action needed. Confirm by running Query 1B.
- **Outcome B:** Two records found — one "Football Academy NJ" record and one "FA 12G Black" record. Merge is needed. Proceed to Query 1C then Query 1D.
- **Outcome C:** No records found — source gap. These teams are not in our system at all. Flag in briefing.

---

### Query 1B: Check team_merges — Are They Already Linked?

```sql
-- Check if either name variant appears as the merged (non-canonical) side of a merge record
-- If Football Academy NJ was already merged into another record, this returns the canonical name
SELECT
  t_canonical.team_id    AS canonical_id,
  t_canonical.team_name  AS canonical_name,
  t_merged.team_id       AS merged_id,
  t_merged.team_name     AS merged_name,
  tm.merge_method,
  tm.verified,
  tm.created_at
FROM team_merges tm
JOIN teams t_canonical ON tm.canonical_team_id = t_canonical.team_id
JOIN teams t_merged    ON tm.merged_team_id    = t_merged.team_id
WHERE
  t_canonical.team_name ILIKE '%Football Academy%'
  OR t_canonical.team_name ILIKE '%FA 12G%'
  OR t_merged.team_name ILIKE '%Football Academy%'
  OR t_merged.team_name ILIKE '%FA 12G%';
```

**Interpreting results:**
- If a row returns with `canonical_name = 'The Football Academy NJ ...'` and `merged_name = 'FA 12G Black'`: already merged. Done — go to Section 2.
- If no rows return: not merged. Proceed to Query 1C.
- If a row returns but the two names appear as separate canonical entries (not linked to each other): fragmented with no merge record. Proceed to Query 1C and 1D.

---

### Query 1C: If Not Merged — Rating Impact Calculation

Run only if Query 1B shows no existing merge record linking the two names.

```sql
-- Get the full record for both records to calculate hypothetical merged rating
SELECT
  t.team_id,
  t.team_name,
  r.rating,
  r.rd,
  r.match_count,
  r.last_game_date,
  r.age_group,
  r.gender
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE
  (t.team_name ILIKE '%Football Academy%' OR t.team_name ILIKE '%FA 12G%')
  AND r.age_group = 'U13'
  AND r.gender = 'F'
ORDER BY r.match_count DESC;
```

**Rating impact estimate (calculate from output):**

Glicko-2 merged rating approximation — weighted by match_count:

```
merged_rating ≈ (rating_A × match_count_A + rating_B × match_count_B)
                ÷ (match_count_A + match_count_B)
```

If `rating_A = 1480` (30 games) and `rating_B = 1350` (12 games):
```
merged_rating ≈ (1480 × 30 + 1350 × 12) ÷ 42 = (44400 + 16200) ÷ 42 ≈ 1443
```

The merged team's rank will be determined by this weighted rating. A team at 1443 = Silver band (≥1350). A team at 1480 = Gold if ≥1500, else still Silver. **If the two fragments combined represent a rating ≥1500, the merged team earns a Gold badge.**

---

### Query 1D: Game Overlap Verification (Confirm They're the Same Team)

Before merging, verify the two records don't have overlapping game dates — overlapping dates on two teams with the same name would mean two different real-world teams (e.g., different age brackets within the same club), not a dedup error.

```sql
-- For each team ID found in Query 1A (substitute actual team_ids)
-- Check date ranges and age group to confirm they're the same real-world team
SELECT
  g.source,
  COUNT(*) AS game_count,
  MIN(g.match_date) AS first_game,
  MAX(g.match_date) AS last_game,
  g.age_group,
  g.gender
FROM games g
WHERE (g.home_team_id = '[TEAM_ID_A]' OR g.away_team_id = '[TEAM_ID_A]')
  AND g.is_hidden = false
GROUP BY g.source, g.age_group, g.gender

UNION ALL

SELECT
  g.source,
  COUNT(*) AS game_count,
  MIN(g.match_date) AS first_game,
  MAX(g.match_date) AS last_game,
  g.age_group,
  g.gender
FROM games g
WHERE (g.home_team_id = '[TEAM_ID_B]' OR g.away_team_id = '[TEAM_ID_B]')
  AND g.is_hidden = false
GROUP BY g.source, g.age_group, g.gender
ORDER BY first_game;
```

**Merge is safe if:**
- Both records have the same age_group and gender
- Game date ranges do not significantly overlap (team A has games Jan–Jun 2025, team B has games Jul–Dec 2025 = same team, different naming period)
- OR both have the same date range but come from different sources (GotSport vs TGS = same games in two sources, already a known dedup pattern)

**Do NOT merge if:**
- Both records have substantial overlapping game dates AND different opponent pools — these are two genuinely different teams with similar names.

---

### Query 1E: Merge Execution SQL (Stage Until Verified)

After confirming safe to merge (Query 1D passes), insert the merge record. The record with more games and the older `created_at` is canonical.

```sql
-- STAGED — do not run until Query 1D confirms safe to merge
-- Substitute canonical_team_id and merged_team_id from Query 1A output

INSERT INTO team_merges (
  canonical_team_id,
  merged_team_id,
  merge_method,
  verified,
  verified_by,
  created_at
) VALUES (
  '[CANONICAL_TEAM_ID]',   -- higher match_count team from Query 1C
  '[MERGED_TEAM_ID]',      -- lower match_count team from Query 1C
  'manual',
  true,
  'ELO_PREMERGE_2026-04-23',
  NOW()
);
```

After inserting the merge record, run a full ranking recompute for U13G:

```bash
node scripts/compute-rankings.js --age-group=U13 --gender=F
# Or if scoped recompute is not supported:
node scripts/compute-rankings.js
```

Post-merge verification — confirm the canonical team now has the expected combined rating:

```sql
SELECT t.team_name, r.rating, r.rank, r.match_count
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE t.team_id = '[CANONICAL_TEAM_ID]'
  AND r.age_group = 'U13' AND r.gender = 'F';
```

---

## Section 2: All 16 USA Rank Reference Teams — Split Candidate Analysis

Run these as a batch. For each team, the query checks for name variants in our teams table. Results feed the action table below.

### Batch Query — All 16 Teams Name Search

```sql
-- U13G Reference Teams
SELECT 'Football Academy NJ' AS ref_name, t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%Football Academy%' OR t.team_name ILIKE '%FA 12G%'
UNION ALL
SELECT 'Wall Soccer Elite Porto', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%Wall%Elite Porto%' OR t.team_name ILIKE '%Wall Soccer%Porto%'
UNION ALL
SELECT 'FC Stars G2012 ECNL White', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%FC Stars%2012%' OR t.team_name ILIKE '%FC Stars%G12%'
UNION ALL
SELECT 'NE Surf G2012 State Navy', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%NE Surf%2012%' OR t.team_name ILIKE '%Surf%G12%State%'
UNION ALL
SELECT 'Scorpions SC ECNL G12', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%Scorpions%G12%' OR t.team_name ILIKE '%Scorpions%ECNL%12%'
UNION ALL
SELECT 'NYCFC Girls 2012 North', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%NYCFC%Girls%2012%' OR t.team_name ILIKE '%NYCFC%12G%'
UNION ALL
SELECT 'Dix Hills EST Angel City', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%Dix Hills%2012%' OR t.team_name ILIKE '%Dix Hills%Angel City%'
UNION ALL
SELECT 'BVB Pittsburgh 12G Prem', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%BVB%Pittsburgh%12%' OR t.team_name ILIKE '%BVB%Pitt%12G%'
UNION ALL
SELECT 'Old Line FC Girls 12 Black', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%Old Line%Girls%12%' OR t.team_name ILIKE '%Old Line%12 Black%'
UNION ALL
SELECT 'SYC 2012G GA II', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.team_name ILIKE '%SYC%2012%G%' OR t.team_name ILIKE '%SYC%GA%12%'
-- U14G Reference Teams
UNION ALL
SELECT 'Maryland Rush Azzurri 2011', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U14' AND r.gender = 'F'
WHERE t.team_name ILIKE '%Maryland Rush%Azzurri%' OR t.team_name ILIKE '%Md Rush%2011%'
UNION ALL
SELECT 'NJ Premier FC G2011', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U14' AND r.gender = 'F'
WHERE t.team_name ILIKE '%NJ Premier%2011%' OR t.team_name ILIKE '%NJ Premier%G11%'
UNION ALL
SELECT 'Marlton Chiefs Premier 2011', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U14' AND r.gender = 'F'
WHERE t.team_name ILIKE '%Marlton%Chiefs%' OR t.team_name ILIKE '%Marlton%2011%'
UNION ALL
SELECT 'Greater Severna Park 11G Green', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U14' AND r.gender = 'F'
WHERE t.team_name ILIKE '%Severna Park%11G%' OR t.team_name ILIKE '%Severna Park%2011%'
UNION ALL
SELECT 'Seacoast United MA 2011G', t.team_id, t.team_name, r.rating, r.rank, r.match_count
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.team_id AND r.age_group = 'U14' AND r.gender = 'F'
WHERE t.team_name ILIKE '%Seacoast%2011%' OR t.team_name ILIKE '%Seacoast United%MA%Elite%'
ORDER BY ref_name, r.rating DESC NULLS LAST;
```

### Action Classification Table

Populate this after running the batch query:

| USA Rank Team | USA Rank # | # Records Found | Potential Split? | Action |
|---|---|---|---|---|
| The Football Academy NJ 2012 Girls Black | #4 | [QUERY] | [YES/NO] | [see Section 1] |
| FA 12G Black | #38 | [QUERY] | [YES — same club as above] | [see Section 1] |
| Wall Soccer Club Wall Elite Porto | #12 | [QUERY] | [QUERY] | [confirmed-single / investigate] |
| FC Stars G2012 ECNL White | #121 | [QUERY] | [QUERY] | [confirmed-single / investigate] |
| NE Surf G2012 State Navy | #198 | [QUERY] | [QUERY] | [expected: not found — source gap] |
| Scorpions SC ECNL G12 | #317 | [QUERY] | [QUERY] | [confirmed-single / investigate] |
| NYCFC Girls 2012 North | #491 | [QUERY] | [QUERY] | [confirmed-single / investigate] |
| Dix Hills EST Angel City G2012 | #562 | [QUERY] | [QUERY] | [expected: not found — source gap] |
| BVB Int'l Academy Pittsburgh 12G Prem | #613 | [QUERY] | [QUERY] | [confirmed-single / investigate] |
| Old Line FC Girls 12 Black | #741 | [QUERY] | [QUERY] | [confirmed-single / investigate] |
| SYC 2012G GA II | #826 | [QUERY] | [QUERY] | [confirmed-single / investigate] |
| Maryland Rush Azzurri 2011 | #144 | [QUERY] | [QUERY] | [confirmed-single / investigate] |
| NJ Premier FC G2011 | #810 | [QUERY] | [QUERY] | [confirmed-single / investigate] |
| Marlton Chiefs Premier 2011 | #926 | [QUERY] | [QUERY] | [expected: not found — source gap] |
| Greater Severna Park 11G Green | #988 | [QUERY] | [QUERY] | [expected: not found — source gap] |
| Seacoast United MA 2011G MA Elite Navy | #1995 | [QUERY] | [QUERY] | [expected: not found — source gap] |

**Action legend:**
- `confirmed-merged`: team_merges record already links the variants
- `confirmed-single`: only one record found, clean identity
- `investigate`: 2+ records found — run date overlap check (Query 1D pattern) before deciding
- `merge-recommended`: investigation confirms same team, pre-written merge SQL below
- `source-gap`: not found at all — not a dedup problem; flag for data roadmap

---

## Section 3: Merge Recommendations

Only Football Academy NJ / FA 12G Black is pre-specified here because it is the highest-priority case. For any `investigate` case found in Section 2 that proves to be a split, apply the same Query 1D + 1E pattern from Section 1.

### Football Academy NJ ↔ FA 12G Black

**Priority: #1 — must resolve before Rank Bands implementation**

Pre-written merge SQL (fill in team IDs from Query 1A):

```sql
-- PRE-MERGE SNAPSHOT (run first, save output)
SELECT
  t.team_id, t.team_name, r.rating, r.rank, r.rd, r.match_count, r.last_game_date
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE (t.team_name ILIKE '%Football Academy%' OR t.team_name ILIKE '%FA 12G%')
  AND r.age_group = 'U13' AND r.gender = 'F';
```

```sql
-- MERGE EXECUTION (staged — substitute IDs from Query 1A)
-- [CANONICAL_ID] = higher match_count, older created_at
-- [MERGED_ID] = lower match_count, newer created_at
INSERT INTO team_merges (
  canonical_team_id,
  merged_team_id,
  merge_method,
  verified,
  verified_by,
  created_at
) VALUES (
  '[CANONICAL_ID]',
  '[MERGED_ID]',
  'manual',
  true,
  'ELO_PREMERGE_2026-04-23',
  NOW()
)
ON CONFLICT DO NOTHING;  -- safe to run twice
```

```sql
-- POST-MERGE VERIFICATION (run after compute-rankings.js completes)
SELECT t.team_name, r.rating, r.rank, r.match_count, r.last_game_date
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE t.team_id = '[CANONICAL_ID]'
  AND r.age_group = 'U13' AND r.gender = 'F';
```

**Expected post-merge result:** combined match_count = A + B; rating = weighted average (will be recalculated by ranking engine). Rank should move meaningfully closer to USA Rank's #4.

---

## Section 4: Pre-Merge Verification Protocol

For any merge in this document:

### Step 1 — Before-State Snapshot (save to file)

```sql
-- Save this output before any merge
\o /tmp/pre-merge-snapshot-[TEAM_NAME]-[DATE].txt
SELECT
  t.team_id, t.team_name, r.rating, r.rank, r.rd, r.match_count,
  r.last_game_date, r.age_group, r.gender,
  (SELECT COUNT(*) FROM games g
   WHERE (g.home_team_id = t.team_id OR g.away_team_id = t.team_id)
     AND g.is_hidden = false) AS visible_game_count
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE t.team_id IN ('[CANONICAL_ID]', '[MERGED_ID]');
\o
```

### Step 2 — Merge Execution

Insert the `team_merges` record (Query 1E above). Run `compute-rankings.js`.

### Step 3 — After-State Verification

```sql
-- Confirm merge took effect
SELECT
  t.team_id, t.team_name, r.rating, r.rank, r.rd, r.match_count
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE t.team_id = '[CANONICAL_ID]'
  AND r.age_group = 'U13' AND r.gender = 'F';

-- Confirm merged team's games are now attributed to canonical
SELECT COUNT(*) AS games_now_under_canonical
FROM games g
WHERE (g.home_team_id = '[CANONICAL_ID]' OR g.away_team_id = '[CANONICAL_ID]')
  AND g.is_hidden = false;
```

Pass criteria:
- `match_count` for canonical = pre-merge canonical count + pre-merge merged count (±1 for edge cases)
- `rating` is within ±50 of the weighted-average estimate from Section 1 Query 1C calculation
- `rank` for canonical is better (lower number) than pre-merge rank

### Step 4 — Rollback Path

If the merged rating is clearly wrong (e.g., the canonical team drops 200+ ranks after merge):

```sql
-- Delete the merge record to restore split state
DELETE FROM team_merges
WHERE canonical_team_id = '[CANONICAL_ID]'
  AND merged_team_id = '[MERGED_ID]'
  AND verified_by = 'ELO_PREMERGE_2026-04-23';
```

Re-run `compute-rankings.js`. Both teams will revert to their pre-merge ratings. The rollback is safe because the merge record is the only thing changed — no game records are modified.

---

## Expected Session Time

Running all queries in psql against `youth_soccer` at localhost:5432:
- Section 1 queries (Football Academy NJ case): ~10 minutes
- Section 2 batch query + action table review: ~10 minutes
- Section 3 merge execution + post-verification: ~10 minutes (if Football Academy case confirms split)
- Total: 30–40 minutes

---

## Related Notes

- [[USA Rank Ranking Accuracy Audit — April 2026]] — Section 2 comparison table references this merge
- [[USA Rank Comparison 2026-04-23]] — companion analysis
- [[Dedup Strategy]] — team_merges logic and merge method types
- [[Database Schema]] — teams and team_merges table schemas
- [[Rank Bands — Implementation Plan]] — the feature this unblocks; Rank Bands must ship after this audit runs
