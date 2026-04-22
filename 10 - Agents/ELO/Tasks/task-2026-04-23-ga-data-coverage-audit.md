---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
status: completed
completed: 2026-04-23
deliverables: "Leagues/GA.md (updated), Reports/ga-data-coverage-audit-2026-04-23.md, Reports/ga-calibration-validation-2026-04-23.md, Queue/pending-2026-04-23-ga-calibration-review.md"
priority: high
---

# Task: Girls Academy (GA) Data Coverage Audit and Calibration Validation

## Context

Strategic Insights Risk 4 and Action Item 5 both flag the same gap: **there is no dedicated Girls Academy (GA) note, no documented GA data source, and no empirical validation of the GA calibration values (140 girls, 100 boys).** The vault references GA over 10 times across League Hierarchy, Recent Changes 2024-2026, Cross-League Analysis, and Competitors — but it has never been formally audited.

ELO's 2026-04-22 briefing also independently flagged this: "Audit the GA (Girls Academy) data coverage gap flagged in Strategic Insights — no dedicated GA note exists despite GA being a Tier 1 league; worth creating one and verifying the 140-point calibration has empirical backing."

This is HIGH priority because:
1. GA is a Tier 1 girls league with 120+ clubs and significant cross-league competition with ECNL
2. The ASPIRE launch (MLS NEXT-GA alliance) creates a new sub-tier that may need its own calibration
3. The calibration values (140 girls, 100 boys) are documented in the ranking engine but their empirical basis is not documented anywhere in the vault

## What to Do

### Step 1: Count GA Games in the Database

Query the `games` table to find all GA-classified games:

```sql
-- Games with GA classification via event_tiers
SELECT COUNT(*) as game_count, COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) as team_count
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name  -- or however event_tier is joined
WHERE et.event_tier IN ('ga', 'girls_academy', 'ga_aspire')
  AND g.is_hidden = false;

-- Also check for GA games via event_name pattern matching
SELECT COUNT(*), event_name
FROM games
WHERE (LOWER(event_name) LIKE '%girls academy%' OR LOWER(event_name) LIKE '%ga aspire%')
  AND is_hidden = false
GROUP BY event_name
ORDER BY COUNT(*) DESC
LIMIT 50;
```

Report:
- Total GA game count in the database
- How many of those games come from GotSport API vs HTML scraper vs other sources
- Age group distribution of GA games (U12 through U19)
- Gender split (GA is primarily girls, but boys GA events exist)
- Date range of GA data (how many seasons are covered)

### Step 2: Validate the GA Girls Calibration (140 Points)

The current GA girls calibration is 140. This is between ECNL girls (130) and MLS NEXT boys (160). Validate it empirically using the same cross-league win rate methodology applied to the MLS NEXT tier split analysis.

**Required cross-league matchups for GA girls:**
- GA girls vs ECNL girls (win rate from GA side)
- GA girls vs ECNL RL girls (win rate from GA side)
- GA girls vs NPL girls (win rate from GA side)

For each matchup, run the same analysis as `Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md` Section 2:
- Minimum 200 game sample per matchup pair
- Win rate = (wins + 0.5 × draws) / total
- Map win rate to implied calibration value using the Elo framework

**Expected finding:** If GA girls (140) beats ECNL girls (130) at roughly 55-60%, the calibration is appropriate. If the win rate is significantly higher or lower, the calibration may need adjustment.

**Required cross-league matchups for GA boys:**
- GA boys vs ECNL boys (win rate from GA side)
- GA boys vs MLS NEXT Academy boys (if sample size allows)

The current GA boys calibration is 100 — between ECNL boys (120) and ECNL RL (55). A win rate of approximately 40-45% vs ECNL boys would validate this.

### Step 3: ASPIRE Tier Analysis

The MLS NEXT-GA ASPIRE launch (documented in Recent Changes 2024-2026) created a new sub-tier within GA. Determine:

1. Are ASPIRE events distinguishable from standard GA events in the current `event_tiers` table? If not, what pattern matching would identify them?
2. How many ASPIRE games exist in the database?
3. Do ASPIRE teams perform differently than standard GA teams in cross-league play? (If ASPIRE is the elite tier of GA, it may warrant a calibration closer to ECNL (130) than standard GA (140). If standard GA is the elite tier, the 140 is for the top level and ASPIRE would be higher.)

Note: The naming is potentially confusing. Clarify in your analysis which is the higher tier — GA core or GA ASPIRE — and what the calibration should be for each.

### Step 4: Create the GA Knowledge Note

Create `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Leagues/GA.md` (or the appropriate location in the vault — check where ECNL.md and MLS NEXT.md live).

The note should follow the same structure as other league notes and include:
- Organization overview: Girls Academy Alliance, founding, current membership count
- Relationship to MLS NEXT (the ASPIRE partnership)
- Data sources: how GA games flow into Evo Draw (primarily via GotSport API)
- Calibration values: boys (100) and girls (140), with empirical validation from Step 2
- ASPIRE tier: calibration value and how it's distinguished in the event_tiers table
- Scope notes: age groups covered, whether boys GA is tracked separately from girls GA
- Cross-league analysis results from Step 2

### Step 5: Flag Any Calibration Adjustments Needed

After completing Steps 1-4, determine:
- Does the GA girls (140) calibration need adjustment based on empirical win rates?
- Does the GA boys (100) calibration need adjustment?
- Does ASPIRE need a separate calibration entry?

If adjustments are needed, prepare a queue item (`Queue/pending-2026-04-23-ga-calibration-review.md`) for Presten's review — do NOT apply calibration changes without approval, following the same process as the Brier score fixes.

## What to Read

- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Strategic Insights.md` — Risk 4, Blind Spot: GA Data Coverage, Action Item 5
- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md` — Section 2 methodology (replicate this for GA)
- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Knowledge/Recent Changes 2024-2026.md` — ASPIRE launch context
- `[[League Hierarchy]]` vault note — current GA calibration values (140 girls, 100 boys)
- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Rankings/Calibration Validation 2026-04-22.md` — Section 6 (updated calibration table) for context on how GA fits in the full hierarchy

## What to Produce

1. `Reports/ga-data-coverage-audit-2026-04-23.md` — database query results: game counts, source breakdown, age group distribution, date coverage
2. `Reports/ga-calibration-validation-2026-04-23.md` — cross-league win rate analysis with comparison table (current calibration vs win-rate implied calibration)
3. A new GA league note in the vault (path to match existing league notes)
4. `Queue/pending-2026-04-23-ga-calibration-review.md` if any calibration adjustments are recommended (leave blank / omit if current values are validated)

## Definition of Done

- GA game count in the database is a known number (not "presumably included via GotSport")
- The 140/100 calibration values either have empirical win-rate backing or a queue item is posted requesting review
- ASPIRE is either distinguished in the event_tiers table or a recommendation exists for how to add it
- The GA knowledge note exists in the vault with the same quality as other league notes

## SENTINEL Note

This gap has existed since the vault was created. GA is a Tier 1 league. The calibration is being applied to thousands of games without empirical validation. This is a data integrity issue, not a nice-to-have. Treat it as HIGH priority accordingly.
