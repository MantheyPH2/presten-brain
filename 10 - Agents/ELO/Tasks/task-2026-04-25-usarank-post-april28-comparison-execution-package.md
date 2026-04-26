---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-05-03
deliverable: "02 - Tiger Tournaments/Projects/Rankings/USA Rank Post-April-28 Comparison — Execution Package.md"
---

# Task: USA Rank Post-April-28 Comparison — Execution Package

## Context

ELO filed `Rankings/USA Rank Delta Interpretation Spec.md` — a framework for understanding what Evo Draw vs USA Rank gaps mean. SENTINEL's mandate includes comparing against competitors for accuracy and completeness. What does not exist: a concrete, executable comparison package ELO runs after April 28, using the post-recompute rankings, to determine whether the GA ASPIRE fix moved Evo Draw's Girls rankings in the direction that closes the competitive gap with USA Rank.

The distinction: the interpretation spec explains what deltas mean in theory. This execution package gives ELO the specific queries to run against the live database, the specific USA Rank reference rankings to compare against, and the specific verdicts to file. It is a one-time post-fix validation, not an ongoing process.

The timing matters: April 28 makes the most significant calibration change to Girls ratings since system inception. If the fix overcorrects (Girls GA teams drop too far) or undercorrects (minimal movement), Evo Draw's competitive positioning vs USA Rank changes in a way SENTINEL and Presten need to know before DSS.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/USA Rank Post-April-28 Comparison — Execution Package.md`

---

### Section 1: Comparison Scope

What this comparison covers and what it does not:
- **In scope:** Girls top-30 rankings by age group (U13G through U17G). Post-April-28 recompute values only.
- **In scope:** Teams that USA Rank prominently features for Girls (ECNL, GA, MLS NEXT-affiliated clubs)
- **Out of scope:** Boys rankings (April 28 should not move Boys — any Boys gap with USA Rank is pre-existing, not April-28-caused)
- **Out of scope:** Individual team-level comparison (too granular for a calibration sanity check)

ELO states the comparison level: age group average and top-decile average. Not team-by-team.

---

### Section 2: Evo Draw Data Pull Queries

Three copy-paste-ready queries to run after April 28 recompute:

**Query 1 — Girls Top-30 by Age Group (post-recompute)**
```sql
-- ELO writes the actual query
-- Returns: age_group, team_name, elo_rating, rank_within_age_group
-- Filter: gender = 'G', rank_within_age_group <= 30
-- Order: age_group ASC, rank_within_age_group ASC
```

**Query 2 — Girls Average Rating by Tier and Age Group**
```sql
-- ELO writes the actual query
-- Returns: age_group, event_tier, avg_rating, team_count
-- Filter: gender = 'G', event_tier IN ('ecnl', 'mlsnext', 'ga_aspire', 'ga')
-- Provides the aggregate comparison point vs USA Rank tier averages
```

**Query 3 — Pre/Post April-28 Delta for Girls GA ASPIRE Teams**
```sql
-- ELO writes the actual query
-- Returns: team_name, age_group, pre_fix_rating, post_fix_rating, delta
-- Requires: pre-April-28 baseline snapshot values from Rankings/Pre-April-28 Baseline Snapshot
-- If baseline snapshot is in a separate table or file: ELO notes the join strategy here
```

---

### Section 3: USA Rank Reference Data

ELO states the USA Rank reference values to compare against. These must be specific enough to make the comparison executable — not "look at USA Rank and compare," but "USA Rank shows GA ASPIRE Girls teams averaging approximately X points at U15G as of [reference date]."

Sources for USA Rank reference values:
- If ELO has USA Rank data in the vault from prior comparison work: cite the file and the relevant figures
- If USA Rank reference values must be looked up: SENTINEL will pull the values and provide them; ELO notes "SENTINEL provides USA Rank reference values for [N] age groups" and formats the table where Presten can fill them in

| Age Group | USA Rank Top-30 Avg (reference) | Evo Draw Top-30 Avg (post-April-28) | Delta | Delta Pre-April-28 | Direction |
|-----------|--------------------------------|--------------------------------------|-------|-------------------|-----------|
| U13G | | [fill from Query 1] | | [from baseline] | |
| U14G | | | | | |
| U15G | | | | | |
| U16G | | | | | |
| U17G | | | | | |

"Direction" column: `converging` (gap narrowed post-fix) / `neutral` (gap unchanged) / `diverging` (gap widened post-fix) / `inverted` (Evo Draw now ahead where USA Rank was ahead, or vice versa)

---

### Section 4: Verdict Framework

ELO defines what each direction outcome means for DSS:

| Outcome | Meaning | DSS Implication |
|---------|---------|----------------|
| Converging | Fix moved GA ASPIRE Girls toward expected range; gap with USA Rank narrowed | Girls calibration narrative is strengthened |
| Neutral | Fix applied but did not significantly change competitive positioning | Calibration is corrected but was not the source of the competitive gap |
| Diverging | Fix moved GA ASPIRE Girls away from USA Rank — possible overcorrection | Flag to SENTINEL; may need cal value review |
| Inverted | Rankings order reversed at category level | Flag immediately — indicates calibration or data error |

ELO writes one paragraph for each outcome explaining what ELO would investigate and what SENTINEL would be told.

---

### Section 5: Comparison Verdict (fill after April 29 analysis)

```
Date comparison run: [date]
Data source: post-April-28 recompute + [Pre-April-28 baseline snapshot reference]
USA Rank reference date: [reference date of USA Rank data used]

Outcome by age group:
  U13G: [converging / neutral / diverging / inverted]
  U14G: [converging / neutral / diverging / inverted]
  U15G: [converging / neutral / diverging / inverted]
  U16G: [converging / neutral / diverging / inverted]
  U17G: [converging / neutral / diverging / inverted]

Overall direction: [majority outcome]
Largest improvement: [age group and delta]
Largest concern: [age group and issue, or "none"]

ELO assessment (2–3 sentences): [plain language verdict on whether the fix moved Evo Draw's Girls rankings in the right direction relative to competitors]

SENTINEL disposition:
Girls calibration competitive positioning: IMPROVED / UNCHANGED / REGRESSED
DSS narrative impact: [one sentence on what this means for the demo claim]
```

---

## Definition of Done

- Execution package filed with all 3 queries written and copy-paste ready
- USA Rank reference table present (either pre-filled or marked "SENTINEL provides values")
- Verdict framework present with DSS implications per outcome
- Verdict Section 5 filled and filed to SENTINEL by May 3 (3 days after April 29 analysis session)
- ELO summary in briefing: "USA Rank post-April-28 comparison complete. Direction: [converging/neutral/diverging]. Girls calibration positioning: [improved/unchanged/regressed] vs competitors."

## Why This Matters

SENTINEL's Config mandates competitor cross-checks. The DSS audience will ask: "How does this compare to USA Rank?" Presten needs a calibrated answer. Without this comparison, the answer is "we believe our rankings are accurate" — a qualitative claim. With this comparison, the answer is "after the GA ASPIRE fix, our Girls U15G average rating moved 22 points closer to USA Rank's benchmark, consistent with correcting a known overcalibration" — a quantitative claim with a documented mechanism. That is the difference between a credible demo claim and a hand-wavy assertion.

## References

- `Rankings/USA Rank Delta Interpretation Spec.md` — interpretation framework this execution package operationalizes
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted magnitudes this comparison tests
- `Rankings/April 29 Analysis — Results Filing Template.md` — actuals source for this comparison
- `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` — pre-fix values for Query 3 delta calculation
- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — methodology context
