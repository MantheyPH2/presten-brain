---
title: Rank Bands — Demo Examples Document
type: demo-preparation
author: ELO
date: 2026-04-25
status: pre-april28-draft
update_trigger: April 29 — confirm band assignments post-fix before May 9
due: 2026-05-07
related_task: task-2026-04-25-rank-bands-demo-examples
related: "[[Rank Bands — Post-April-28 Validation Spec]]", "[[Rank Bands Threshold Validation — 2026-04-24]]", "[[DSS Demo — Feature Sequence Plan]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]"
tags: [rank-bands, dss, demo, examples, evo-draw]
---

# Rank Bands — Demo Examples Document

> **Purpose:** Pre-select named teams and queries for the Rank Bands DSS demo. Eliminates live improvisation; gives Presten a reliable script with known team names and a compelling narrative arc.
>
> **Band naming note:** The implementation uses 6 bands: Gold, Silver, Bronze, Red, Blue, Green. This document uses those names. The task spec used placeholder names (Elite, High, Mid, Developing, Low) — superseded by the canonical implementation names.
>
> **Primary demo age group:** U15G and U16G (highest demo value per `DSS Demo — Feature Sequence Plan.md`). U13G used for Football Academy NJ sequences only.
>
> **Update required:** After April 28 recompute, run the confirmation queries in Section 5. If any primary example shifts bands, activate the backup.
>
> **SENTINEL review:** Before May 9 readiness assessment.

---

## Section 1: Band Overview Query (Demo Opening Shot)

Run at the start of the Rank Bands demo to show the audience the full distribution. Establishes that Rank Bands is a real distribution across thousands of teams — not a label applied to a small list.

```sql
-- Rank Bands: full distribution by age group and gender
SELECT
  rb.age_group,
  rb.gender,
  rb.band,
  COUNT(*) AS team_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (
    PARTITION BY rb.age_group, rb.gender
  ), 1) AS pct_of_age_group
FROM rankings_with_bands rb
ORDER BY
  rb.age_group,
  rb.gender,
  CASE rb.band
    WHEN 'Gold'   THEN 1
    WHEN 'Silver' THEN 2
    WHEN 'Bronze' THEN 3
    WHEN 'Red'    THEN 4
    WHEN 'Blue'   THEN 5
    WHEN 'Green'  THEN 6
  END;
```

**Demo purpose:** Show the audience: "There are [N] teams in the Gold band across Girls age groups — and [M] teams in total. The band doesn't mean 'not in top 100' — it means we've measured where you are across every ranked program." The distribution percentage confirms Gold is rare (~1%) and meaningful.

**Suggested opening line:** "Let me show you the full picture first. We rank [total_teams] Girls U13 through U17 programs. Every one of them gets a band."

---

## Section 2: Named Examples by Band

Selected from ELO's knowledge of the current Girls rankings. All examples are U15G (primary demo age group) with U16G backups. Ratings are pre-April-28 estimates; confirm against DB using Section 3 query after April 29.

**Selection criteria used:**
1. Club name is recognizable to a NE-corridor soccer audience (ECNL programs, strong regional clubs)
2. Band assignment is clearly correct — not a borderline case
3. Team has a meaningful game count (≥ 15 games) so the band is data-confident
4. Primary examples avoid high GA ASPIRE exposure (post-April-28 stability, per Section 5)

| Band | Team Name | Age Group | Expected Rating Range | Why This Is a Good Example |
|------|-----------|-----------|----------------------|---------------------------|
| Gold | Football Academy NJ | U13G | ≥ 1500 | ECNL team, USA Rank #4 equivalent — the credibility anchor; audience will recognize it |
| Gold | PDA (NJ) | U15G | ≥ 1500 | Top-5 Girls club in the NE corridor; multi-age-group ECNL presence |
| Silver | FC Stars of Massachusetts | U15G | 1350–1499 | Well-known NE ECNL program; clear step below top elite |
| Silver | Cedar Stars Academy | U16G | 1350–1499 | NJ-based; strong ECNL RL / competitive program |
| Bronze | PA Classics | U15G | 1200–1349 | PA ECNL program, primary Club Rankings reference club — continuity across demo features |
| Bronze | NJ Wildcats | U15G | 1200–1349 | NJ-based, recognizable to regional audience; solid competitive program |
| Red | Match Fit Academy | U15G | 1050–1199 | NJ club; strong state-level program, well-known but below ECNL caliber |
| Red | Sporting NJ | U16G | 1050–1199 | Regional competitor; represents the well-organized mid-tier club |
| Blue | NJ Stallions | U15G | 900–1049 | State-league level; legitimate active program with consistent game history |
| Blue | Delaware FC | U16G | 900–1049 | Regional mid-tier; represents programs outside top-100 that still rank |
| Green | [Local/State program — identify by query] | U15G | < 900 | Run lookup query (Section 3) for a local club the audience knows — reinforces coverage breadth |
| Green | [Backup — identify by query] | U16G | < 900 | Backup; confirm via Section 3 |

> **Green band note:** ELO does not pre-select a specific Green team by name because Green programs are less nationally recognizable and the best example depends on the DSS audience's geography. Presten should run the Section 3 query filtered to Green band and pick a club the audience would know locally. The demo point is not the club name — it's that "even your community club has a number."

---

## Section 3: Individual Team Lookup Query

Run this during the demo to pull up any named team on demand. Copy-paste ready.

```sql
-- Rank Bands: individual team lookup (fuzzy match)
-- Replace 'Football Academy' with any team name fragment
SELECT
  t.name AS team_name,
  rb.age_group,
  rb.gender,
  rb.rating::int AS current_rating,
  rb.band,
  rb.within_band_rank,
  rb.rank AS overall_rank,
  (
    SELECT COUNT(*) FROM games g
    WHERE (g.home_team_id = t.id OR g.away_team_id = t.id)
      AND g.is_hidden = false
  ) AS game_count
FROM rankings_with_bands rb
JOIN teams t ON t.id = rb.team_id
WHERE t.name ILIKE '%Football Academy%'
  AND rb.age_group = 'U13'
  AND rb.gender = 'F'
ORDER BY rb.rating DESC;
```

**To use for any demo team:** Replace `'%Football Academy%'` with `'%[club name fragment]%'` and update `age_group` and `gender` as needed.

**Output for the audience:**
- `team_name` — full team name as it appears in the system
- `current_rating` — the Elo number behind the band
- `band` — the label
- `within_band_rank` — their rank within the band (Gold #1 = top Gold team in their age group)
- `overall_rank` — where they sit across all teams in the age group
- `game_count` — data confidence signal; state "this team has [N] games in our system"

**Demo context note:** A `game_count` of 25+ signals high data confidence. If game_count is < 5, do not use this as a primary example — the band is provisional.

---

## Section 4: Narrative Script

### Scene 1 — Introducing Rank Bands (30 seconds)

"A raw Elo number tells you where a team is ranked, but it doesn't tell you what that *means*. Is rank #47 elite? Regional powerhouse? Solid development program? Depends entirely on the age group and where the distribution breaks. Rank Bands solve that. Instead of 'you're ranked 47th,' you can say 'you're in the Silver band' — which means top 2 to 8 percent of all ranked programs in your age group. The band is meaningful regardless of whether there are 800 or 2,000 teams in the pool."

---

### Scene 2 — The Overview Shot (60 seconds)

*[Run Section 1 query. Wait for results.]*

"Look at the Gold column — [N] teams in U13 Girls Gold. That's roughly 1 percent. That's who we're talking about when we say 'elite' — not top 100, not top 50, the genuine national-caliber programs. Silver is your top 5 to 8 percent — strong ECNL programs that would be competitive at any national event. Bronze is where most ECNL RL and NPL programs land — legitimately competitive, nationally recognized.

Then Red and below covers the majority of teams, and that's actually the most important part for coverage: these programs have ratings. These programs have data. USA Rank doesn't have most of them. We do. A club director who's been wondering 'where do we *actually* stand compared to everyone?' — we can answer that question for Bronze and Red programs the same way we can for Gold."

---

### Scene 3 — The Named Example (90 seconds)

*[Run Section 3 lookup for Football Academy NJ (U13G Gold) and PA Classics (U15G Bronze).]*

"Let me show you what this looks like on a specific team. Football Academy NJ — USA Rank has them at number 4 in U13 Girls. We agree they're elite. Here's the Elo behind that: [rating]. Gold band. Within Gold, they're ranked [within_band_rank]. That's the math behind the badge.

Now let me show you something more interesting — PA Classics. Bronze band, U15 Girls. [rating]. They're not USA Rank top 10. They're not in an ECNL full program. But they have [game_count] games in our system — more data coverage than many ECNL programs get in other systems. A coach at PA Classics can look at this and say: 'We're top 15 to 20 percent. We've earned the Bronze. Our path to Silver means closing a [Silver avg - PA Classics rating] point gap.' That's actionable intelligence. USA Rank doesn't give it to them."

---

## Section 5: Contingency — Post-April-28 Band Stability

The April 28 GA ASPIRE fix raises Girls GA ASPIRE team ratings (see `Post-April-28 Expected Rating Shift — Pre-Run Model.md`). If a primary demo team has significant GA ASPIRE game exposure, their band assignment may shift.

**GA ASPIRE exposure assessment for primary examples:**

| Team | GA ASPIRE Exposure | Band Stability | Action if Shifts |
|------|-------------------|----------------|-----------------|
| Football Academy NJ (U13G) | Low — ECNL team, limited GA ASPIRE games | ✅ Stable | N/A |
| PDA U15G | Low — ECNL team, limited GA ASPIRE games | ✅ Stable | N/A |
| FC Stars of MA U15G | Low — ECNL, GA ASPIRE unlikely | ✅ Stable | Use Cedar Stars backup |
| Cedar Stars Academy U16G | Low — ECNL RL, limited ASPIRE exposure | ✅ Stable | Use backup query |
| PA Classics U15G | Low-Medium — may have some GA ASPIRE exposure | ⚠️ Monitor | Use NJ Wildcats backup |
| NJ Wildcats U15G | Low — state/NPL level, unlikely GA ASPIRE | ✅ Stable | N/A |

**Post-April-28 confirmation query:**

Run this after April 29 to verify all primary examples are in their expected bands before May 9:

```sql
-- Confirm band assignments for primary demo examples post-April-28
SELECT
  t.name AS team_name,
  rb.age_group,
  rb.rating::int AS rating,
  rb.band,
  rb.within_band_rank,
  rb.rank AS overall_rank
FROM rankings_with_bands rb
JOIN teams t ON t.id = rb.team_id
WHERE (
  t.name ILIKE '%Football Academy%'
  OR t.name ILIKE '%PDA%'
  OR t.name ILIKE '%FC Stars%'
  OR t.name ILIKE '%Cedar Stars%'
  OR t.name ILIKE '%PA Classics%'
  OR t.name ILIKE '%NJ Wildcats%'
  OR t.name ILIKE '%Match Fit%'
)
  AND rb.gender = 'F'
  AND rb.age_group IN ('U13', 'U15', 'U16')
ORDER BY rb.age_group, rb.rating DESC;
```

**If any primary example is in a different band than expected:** Use the backup in Section 2. Do not attempt to explain band movement live at the demo — pick a stable example.

---

## Related

- [[Rank Bands — Post-April-28 Validation Spec]] — validation methodology to run April 29
- [[Rank Bands Threshold Validation — 2026-04-24]] — band boundary thresholds
- [[DSS Demo — Feature Sequence Plan]] — demo sequencing context; Section 3 Step 1 is Rank Bands
- [[Post-April-28 Expected Rating Shift — Pre-Run Model]] — which teams may shift bands post-April-28

*ELO — 2026-04-25*
