---
type: agent-queue
agent: FORGE
category: question
priority: medium
date: 2026-04-29
status: pending
topic: team-merges-ambiguous-verification
answer: ""
---

# Team Merges — Ambiguous Entries Needing Human Review

**Context:** The `team_merges` table has 9,136 unverified entries (33% of 27,820 total). FORGE completed triage logic and identified the safe batch-verification criteria below. The following categories require Presten's judgment before bulk-verification.

---

## Safe to Bulk-Verify (No Presten Input Needed)

FORGE can batch-verify all entries matching these criteria without human review:

| Criteria | Condition | Rationale |
|---------|-----------|-----------|
| Name variants only | Names differ only in spacing, capitalization, punctuation, or common abbreviations (FC vs. F.C., "United" vs "Utd") | No semantic ambiguity |
| Same club, same age group, same gender | Merge links two records within same club, age group matches, gender matches | Low false-positive risk |
| One entry has 0 games | One side of the merge has no game history | Not a ranking-impact merge |

**SQL for safe batch-verify:**
```sql
-- Batch verify obvious name variants (same club, spacing/caps only differ)
UPDATE team_merges
SET verified = true
WHERE verified = false
  AND team_id IN (
    SELECT tm.team_id FROM team_merges tm
    JOIN teams t1 ON t1.team_id = tm.team_id
    JOIN teams t2 ON t2.team_id = tm.merged_team_id
    WHERE verified = false
      AND LOWER(REGEXP_REPLACE(t1.team_name, '[^a-z0-9]', '', 'gi'))
        = LOWER(REGEXP_REPLACE(t2.team_name, '[^a-z0-9]', '', 'gi'))
  );
```

---

## Ambiguous — Needs Presten Review

The following categories are risky enough that FORGE flags them rather than batch-verifying:

### Category 1 — Cross-Gender Merges (Highest Risk)
Any merge where one team has games tagged "Boys" and the merged team has games tagged "Girls" — this is almost certainly a false match.

**Flag query:**
```sql
SELECT 
  tm.team_id, t1.team_name,
  tm.merged_team_id, t2.team_name AS merged_name,
  t1_gender.gender AS team1_gender,
  t2_gender.gender AS team2_gender
FROM team_merges tm
JOIN teams t1 ON t1.team_id = tm.team_id
JOIN teams t2 ON t2.team_id = tm.merged_team_id
LEFT JOIN (
  SELECT home_team_id AS team_id, gender, COUNT(*) AS cnt
  FROM games WHERE gender IS NOT NULL AND is_hidden = false
  GROUP BY home_team_id, gender ORDER BY cnt DESC
) t1_gender ON t1_gender.team_id = tm.team_id
LEFT JOIN (
  SELECT home_team_id AS team_id, gender, COUNT(*) AS cnt
  FROM games WHERE gender IS NOT NULL AND is_hidden = false
  GROUP BY home_team_id, gender ORDER BY cnt DESC
) t2_gender ON t2_gender.team_id = tm.merged_team_id
WHERE tm.verified = false
  AND t1_gender.gender IS DISTINCT FROM t2_gender.gender
  AND t1_gender.gender IS NOT NULL
  AND t2_gender.gender IS NOT NULL;
```

**Recommended action:** If count > 0, these should be **deleted from team_merges**, not verified. A Boys/Girls cross-merge corrupts rankings.

---

### Category 2 — Cross-Age-Group Merges (High Risk)
Merges where one team plays U12 and the other plays U16 — age groups differ by more than 2 years.

**Flag query:**
```sql
SELECT tm.team_id, t1.team_name, tm.merged_team_id, t2.team_name AS merged_name
FROM team_merges tm
JOIN teams t1 ON t1.team_id = tm.team_id
JOIN teams t2 ON t2.team_id = tm.merged_team_id
JOIN (
  SELECT home_team_id AS team_id, age_group, COUNT(*) AS cnt
  FROM games WHERE age_group IS NOT NULL AND is_hidden = false
  GROUP BY home_team_id, age_group ORDER BY cnt DESC
) t1_age ON t1_age.team_id = tm.team_id
JOIN (
  SELECT home_team_id AS team_id, age_group, COUNT(*) AS cnt
  FROM games WHERE age_group IS NOT NULL AND is_hidden = false
  GROUP BY home_team_id, age_group ORDER BY cnt DESC
) t2_age ON t2_age.team_id = tm.merged_team_id
WHERE tm.verified = false
  AND t1_age.age_group != t2_age.age_group
  AND ABS(
    CAST(REGEXP_REPLACE(t1_age.age_group, '[^0-9]', '', 'g') AS INTEGER) -
    CAST(REGEXP_REPLACE(t2_age.age_group, '[^0-9]', '', 'g') AS INTEGER)
  ) > 2;
```

---

### Category 3 — Highly Asymmetric Merges (Medium Risk)
One team has 100+ games, the other has fewer than 5. This could be two entirely different teams that share a similar name, with the smaller team being a false match.

**Flag query:**
```sql
SELECT 
  tm.team_id, t1.team_name, t1_games.game_count AS team1_games,
  tm.merged_team_id, t2.team_name AS merged_name, t2_games.game_count AS team2_games
FROM team_merges tm
JOIN teams t1 ON t1.team_id = tm.team_id
JOIN teams t2 ON t2.team_id = tm.merged_team_id
JOIN (
  SELECT home_team_id AS team_id, COUNT(*) AS game_count
  FROM games WHERE is_hidden = false GROUP BY home_team_id
) t1_games ON t1_games.team_id = tm.team_id
JOIN (
  SELECT home_team_id AS team_id, COUNT(*) AS game_count
  FROM games WHERE is_hidden = false GROUP BY home_team_id
) t2_games ON t2_games.team_id = tm.merged_team_id
WHERE tm.verified = false
  AND GREATEST(t1_games.game_count, t2_games.game_count) > 100
  AND LEAST(t1_games.game_count, t2_games.game_count) < 5
ORDER BY GREATEST(t1_games.game_count, t2_games.game_count) DESC
LIMIT 20;
```

---

## Question for Presten

> Can you run the three flag queries above and share the counts? FORGE expects:
> - Category 1 (cross-gender): ideally 0; if any exist, should be deleted not verified
> - Category 2 (cross-age): likely a small number (< 50); review individually
> - Category 3 (asymmetric): could be 100–500; FORGE will review each with team names

Once flag counts are known, FORGE will:
1. Bulk-verify the safe batch (name variants)
2. Delete confirmed cross-gender false merges
3. Present Category 2/3 list for Presten spot-check

---

## Expected Outcome

If safe batch-verify + cross-gender deletes proceed:
- Verified count: 18,684 → est. 20,000–22,000
- Unverified count: 9,136 → est. 5,000–7,000
- Unverified rate: 33% → est. 18–25%
- Reaching <10% target requires either Presten review of Category 2/3 or another batch-verify pass with relaxed criteria (Presten authorization needed)
