---
title: USA Rank Post-April-28 Comparison — Execution Package
type: execution-package
author: ELO
date: 2026-04-25
status: ready-for-execution
due: 2026-05-03
related: "[[USA Rank Delta Interpretation Spec]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[USA Rank Comparison 2026-04-23]]"
tags: [usa-rank, comparison, girls, calibration, april-28, validation, evo-draw]
---

# USA Rank Post-April-28 Comparison — Execution Package

> **Purpose:** One-time post-fix validation. Run after April 28 recompute to determine whether the GA ASPIRE fix moved Evo Draw's Girls rankings in the direction that closes the competitive gap with USA Rank.
>
> **Run date:** After April 29 analysis session (April 29 or 30). **File verdict by May 3.**
>
> This is not an ongoing process. Queries run once against the post-recompute live database. Verdict is filed once.

---

## Section 1: Comparison Scope

**In scope:**
- Girls top-30 rankings by age group: U13G, U14G, U15G, U16G, U17G
- Post-April-28 recompute ratings only (not pre-fix values, except where delta comparison is explicit)
- Teams that USA Rank prominently features (ECNL, GA, MLS NEXT-affiliated clubs)
- Comparison at the **age group average and top-decile average** level — not team-by-team

**Out of scope:**
- Boys rankings. April 28 should not move Boys (cal=100 → 100 for Boys GA). Any Boys gap with USA Rank is pre-existing, not April-28-caused.
- Individual team-level comparison (too granular for a calibration sanity check; team-level comparison is covered in `USA Rank Comparison 2026-04-23.md`)
- Age groups outside U13G–U17G

**Comparison reference from pre-April-28 data:** Query 3 requires the pre-April-28 baseline snapshot from `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md`. If baseline values are in a separate snapshot table, Query 3 joins to it. If baseline values are in a flat file, manual cross-reference is acceptable.

---

## Section 2: Evo Draw Data Pull Queries

Run all three queries after the April 28 recompute. All three require the live `youth_soccer` database.

---

### Query 1 — Girls Top-30 by Age Group (post-recompute)

```sql
-- Girls top-30 per age group, post-April-28 recompute
SELECT
  t.age_group,
  t.team_name,
  r.rating AS elo_rating,
  r.rank   AS rank_within_age_group,
  r.match_count,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS rank_band
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE t.gender = 'F'
  AND t.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND r.rank <= 30
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
ORDER BY t.age_group ASC, r.rank ASC;
```

_Use this to fill the "Evo Draw Top-30 Avg (post-April-28)" column in Section 3's comparison table. Compute avg_rating as `AVG(elo_rating)` for each age group's returned rows._

---

### Query 2 — Girls Average Rating by Tier and Age Group

```sql
-- Girls average rating by tier and age group, post-recompute
SELECT
  t.age_group,
  et.event_tier,
  ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
  COUNT(DISTINCT t.team_id)        AS team_count,
  ROUND(PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY r.rating)::numeric, 1) AS top_10pct_rating
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
JOIN (
  -- join teams to their primary event tier via most recent event
  SELECT g.home_team_id AS team_id, ev.event_tier
  FROM games g
  JOIN events ev ON ev.event_id = g.event_id
  WHERE g.game_date >= NOW() - INTERVAL '12 months'
  UNION
  SELECT g.away_team_id, ev.event_tier
  FROM games g
  JOIN events ev ON ev.event_id = g.event_id
  WHERE g.game_date >= NOW() - INTERVAL '12 months'
) et ON et.team_id = t.team_id
WHERE t.gender = 'F'
  AND t.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND et.event_tier IN ('ecnl', 'mlsnext', 'ga_aspire', 'ga')
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
GROUP BY t.age_group, et.event_tier
ORDER BY t.age_group ASC, et.event_tier ASC;
```

_Use this to verify tier-level separation. The GA ASPIRE cohort should now show avg_rating closer to ECNL than before the fix. If `ga_aspire` avg_rating is within 30 pts of `ecnl` avg_rating at U13G/U14G, convergence is occurring as predicted._

---

### Query 3 — Pre/Post April-28 Delta for Girls GA ASPIRE Teams

```sql
-- Pre/post delta for Girls GA ASPIRE-affiliated teams
-- Requires: pre-April-28 baseline snapshot in table 'ratings_snapshot_pre_april28'
-- If baseline is in a flat file, cross-reference manually by team_id

SELECT
  t.team_name,
  t.age_group,
  snap.rating  AS pre_fix_rating,
  r.rating     AS post_fix_rating,
  (r.rating - snap.rating) AS delta,
  r.rank       AS current_rank
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
JOIN ratings_snapshot_pre_april28 snap ON snap.team_id = t.team_id
WHERE t.gender = 'F'
  AND t.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND EXISTS (
    -- team played in a GA ASPIRE event in the past 18 months
    SELECT 1
    FROM games g
    JOIN events ev ON ev.event_id = g.event_id
    WHERE (g.home_team_id = t.team_id OR g.away_team_id = t.team_id)
      AND ev.event_tier = 'ga_aspire'
      AND g.game_date >= '2025-02-01'
  )
ORDER BY ABS(r.rating - snap.rating) DESC
LIMIT 100;

-- If ratings_snapshot_pre_april28 does not exist, use the pre-April-28 baseline
-- snapshot file from: Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md
-- Cross-reference by team_id or team_name to compute delta manually.
```

_Use this to confirm the fix applied to the right teams. GA ASPIRE-affiliated Girls teams should show positive delta (+5 to +40 pts for most, depending on ASPIRE game share). Teams with zero ASPIRE history should show delta ≈ 0 ± 5 pts._

---

## Section 3: USA Rank Reference Data

_USA Rank reference values for Girls by age group. Source: ELO estimates from prior comparison work (`USA Rank Comparison 2026-04-23.md`) and structural analysis. SENTINEL confirms or provides specific current USA Rank values if available._

**Known USA Rank reference teams (from prior comparison):**

| Team | USA Rank | Age Group | Expected Evo Draw Post-Fix |
|------|----------|-----------|---------------------------|
| Football Academy NJ 2012 Girls Black | #4 | U13G | Top 20 (Gold/Silver) |
| FC Stars G2012 ECNL White | #121 | U13G | Silver |
| SYC 2012G GA II | #826 | U13G | Should move closer to top after GA ASPIRE fix |
| Maryland Rush Azzurri 2011 | #144 | U14G | Silver/Bronze |

**Age group average comparison table:**

| Age Group | USA Rank Top-30 Avg (reference) | Evo Draw Top-30 Avg (post-April-28) | Delta (Evo − USA) | Delta Pre-April-28 | Direction |
|-----------|--------------------------------|--------------------------------------|-------------------|--------------------|-----------|
| U13G | SENTINEL provides | [fill from Query 1] | | [from baseline] | |
| U14G | SENTINEL provides | [fill from Query 1] | | [from baseline] | |
| U15G | SENTINEL provides | [fill from Query 1] | | [from baseline] | |
| U16G | SENTINEL provides | [fill from Query 1] | | [from baseline] | |
| U17G | SENTINEL provides | [fill from Query 1] | | [from baseline] | |

**Direction column key:** `converging` (Evo Draw top-30 avg moved closer to USA Rank avg post-fix) / `neutral` (gap unchanged) / `diverging` (gap widened) / `inverted` (Evo Draw now ahead where USA Rank was ahead, or vice versa)

**Note on USA Rank methodology:** USA Rank uses last 20 games with recency weighting and goal margin. Evo Draw uses full game history via Glicko-2. A persistent gap between the two systems is expected and acceptable — the question is whether the GA ASPIRE fix narrowed the gap, not whether the gap disappears.

---

## Section 4: Verdict Framework

| Outcome | Meaning | DSS Implication |
|---------|---------|----------------|
| **Converging** | The GA ASPIRE fix moved Girls GA ratings toward their correct level; Evo Draw's top-30 avg moved closer to USA Rank's benchmark in the affected age groups | Girls calibration narrative is strengthened. Demo claim: "We corrected a known overcalibration — our Girls ratings now align with competitor benchmarks on the teams we share." |
| **Neutral** | Fix applied but did not significantly change competitive positioning vs. USA Rank | Calibration was corrected, but the competitive gap with USA Rank was not driven by the GA ASPIRE issue. The fix is still correct; the source of the gap is elsewhere (likely source coverage or recency weighting). No change to demo claims. |
| **Diverging** | Fix moved GA ASPIRE Girls away from USA Rank — possible overcorrection | Flag to SENTINEL. ELO would investigate: (a) did the fix overcorrect (magnitude too large), or (b) did USA Rank move between the reference date and April 28? If overcorrection: note for calibration review post-DSS. If USA Rank moved: accept gap as expected. Do not cite GA ASPIRE improvement in DSS demo until investigation clears. |
| **Inverted** | Rankings order reversed at category level — Evo Draw top-30 avg crossed from one side of USA Rank avg to the other | Flag immediately. Either the fix was applied to the wrong population, or USA Rank reference data is outdated. Do not demo GA ASPIRE-related accuracy claims until this is resolved. Investigation takes priority over demo preparation. |

**ELO investigation approach for Diverging/Inverted:**
1. Check if USA Rank reference data is current (if reference is > 60 days old, recency effects may explain apparent divergence)
2. Check Query 3 delta distribution — if GA ASPIRE teams moved > +40 pts avg, overcorrection is likely
3. If overcorrection confirmed: recommend a calibration value re-check post-DSS (not pre-DSS — the fix was needed; the question is fine-tuning)

---

## Section 5: Comparison Verdict

_Fill after April 29 analysis session. File to SENTINEL by May 3._

```
Date comparison run: ___________
Data source: post-April-28 recompute (rankings table) + Pre-April-28 baseline snapshot
USA Rank reference date: [date of USA Rank data used, or "SENTINEL-provided [date]"]

Outcome by age group:
  U13G: [converging / neutral / diverging / inverted]
  U14G: [converging / neutral / diverging / inverted]
  U15G: [converging / neutral / diverging / inverted]
  U16G: [converging / neutral / diverging / inverted]
  U17G: [converging / neutral / diverging / inverted]

Overall direction: [majority outcome across age groups]
Largest improvement: [age group and delta]
Largest concern: [age group and issue, or "none"]

ELO assessment (2–3 sentences): ___________

SENTINEL disposition:
Girls calibration competitive positioning: IMPROVED / UNCHANGED / REGRESSED
DSS narrative impact: ___________
```

---

## Definition of Done

- All 3 queries run and outputs recorded
- USA Rank reference table populated (either pre-filled by SENTINEL or marked "reference pending")
- Section 5 verdict filled and filed by May 3 EOD
- ELO summary in next briefing after results: "USA Rank post-April-28 comparison complete. Direction: [converging/neutral/diverging]. Girls calibration positioning: [improved/unchanged/regressed]. SENTINEL disposition: ___."

---

## References

- `Rankings/USA Rank Delta Interpretation Spec.md` — interpretation framework this package operationalizes
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted GA ASPIRE shift magnitudes
- `Rankings/April 29 Analysis — Results Filing Template.md` — actuals source to cross-reference
- `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` — pre-fix values for Query 3 delta
- `Rankings/USA Rank Comparison 2026-04-23.md` — prior comparison work; reference team list and USA Rank rankings

*ELO — 2026-04-25*
