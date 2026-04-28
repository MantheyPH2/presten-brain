---
title: Event Strength Phase 1 — Implementation Spec
type: implementation-spec
author: ELO
date: 2026-04-27
status: ready-pending-authorization
priority: high
due: 2026-05-05
tags: [event-strength, phase-1, calibration, k-factor, evo-draw, implementation]
related: "[[Event Strength Phase 1 — SENTINEL Authorization Criteria]]", "[[Event Strength Phase 1 — Data Coverage Spot-Check]]", "[[Event Strength Phase 1 — Rankings Impact Preview]]", "[[League Hierarchy]]", "[[Post-Fix Calibration Stability Monitoring Spec]]"
---

# Event Strength Phase 1 — Implementation Spec

> **Status:** Template ready. Execution pending SENTINEL authorization (expected after April 29 G1–G4 pass + May 5 calibration stability confirmation).
>
> When SENTINEL authorizes Phase 1: open this document, run Section 5 pre-flight, then execute Section 4 SQL in sequence.

---

## Section 1: Phase 1 Scope

### In Scope

- All currently active event tiers with calibrated values in the League Hierarchy
- Event strength applied as a **K-factor multiplier** at rating computation time (`compute-rankings.js` Phase 3/Phase 8), not stored as a separate display score
- Multiplier derived from existing League Hierarchy calibration values via the formula in Section 3
- Both Girls and Boys age groups are in scope for Phase 1, **subject to Boys calibration status**: if Boys Option A fails by May 5, Phase 1 applies Girls age groups only and Boys ratings continue with K_base = 1.0× (no multiplier) until Boys calibration is confirmed
- All currently ingested seasons (no historical cutoff — every game in the DB is re-weighted on recompute)

### Out of Scope for Phase 1

| Excluded Item | Reason | Target Phase |
|---------------|--------|-------------|
| Display event strength scores (avg team rating, spread, upset rate) | Separate metric; no circular-dependency risk needed now | Phase 2 |
| MLS NEXT Homegrown/Academy tier split multipliers | MLS NEXT tier split implementation is May 18–20; Phase 1 uses unified mlsnext value | Post-DSS |
| SnapSoccer event tier | Not yet ingested | Phase 2 |
| Manual event strength overrides | Out of scope until display metrics exist | Phase 2 |
| State-level weighting adjustments | Phase 2 feature | Phase 2 |
| Tournament prestige adjustments | Phase 2 feature | Phase 2 |

**Phase 0 → Phase 1 change in plain language:** Phase 0 applies League Hierarchy calibration values uniformly (the same calibration value for every game in a tier, with no K-factor differential based on game-level event quality). Phase 1 formalizes this into explicit K-factor multipliers per event_tier, creating a structured multiplier table that is versioned, auditable, and revisable without code changes to the ELO formula itself.

---

## Section 2: League Tier → Strength Multiplier Mapping

### Derivation Formula

```
multiplier = LEAST(1.5, GREATEST(0.5, cal_value / 100.0))
```

This formula:
- Produces 1.0× at cal=100 (the anchor point — Boys GA and Girls GA ASPIRE post-fix)
- Caps at 1.5× (applied to MLS NEXT at cal=160) to prevent extreme rating volatility
- Floors at 0.5× (applied to sub-25 tiers — local/recreational) to prevent excessive dampening
- Scales linearly between floor and cap

**Rationale for cap at 1.5×:** A game with K_effective = 1.5 × K_base means high-tier wins are worth 50% more than the baseline rate. Exceeding 1.5× risks rating inflation for teams with heavy ECNL/MLS NEXT schedules and makes new-team convergence slower. The 1.5× cap is conservative and reversible.

**Rationale for floor at 0.5×:** Games should always contribute some signal, even at recreational tiers. Flooring at 0.5× ensures a recreational win still moves the needle — just half as much as a mid-tier win.

### Boys Tier Multiplier Table

| event_tier | League / Event Type | Cal Value | Multiplier (Formula) | Final Multiplier | Rationale |
|-----------|--------------------|-----------|--------------------|-----------------|-----------|
| mlsnext | MLS NEXT (pre-split, unified) | 160 | LEAST(1.5, 160/100) = 1.5 | **1.50×** | Boys elite; cap applies |
| mlsnext_homegrown | MLS NEXT Homegrown (post-May-18) | 160 | 1.50× | **1.50×** | Same as unified; cap applies |
| mlsnext_academy | MLS NEXT Academy (post-May-18) | 135 | 135/100 = 1.35 | **1.35×** | Below Homegrown as validated |
| ecnl | ECNL Boys | 120 | 120/100 = 1.20 | **1.20×** | Boys Tier 1 #2 |
| ga | GA Boys | 100 | 100/100 = 1.00 | **1.00×** | Baseline anchor |
| ga_aspire | GA ASPIRE Boys | 100 | 100/100 = 1.00 | **1.00×** | Same as Boys GA (unvalidated split) |
| ecnl_rl | ECNL Regional League | 55 | 55/100 = 0.55 | **0.55×** | Tier 2 peer-validated |
| npl | National Premier League | 55 | 55/100 = 0.55 | **0.55×** | Tier 2 peer-validated |
| dpl | Development Players League | 55 | 55/100 = 0.55 | **0.55×** | Tier 2 peer-validated |
| pre_ecnl | Pre-ECNL | 25 | GREATEST(0.5, 25/100) = 0.5 | **0.50×** | Tier 3; floor applies |
| local | Local/recreational | 20 | GREATEST(0.5, 20/100) = 0.5 | **0.50×** | Recreational; floor applies |
| unknown | Unclassified / NULL | 20 | 0.50× | **0.50×** | Conservative; flag for FORGE to resolve |

### Girls Tier Multiplier Table

| event_tier | League / Event Type | Cal Value | Multiplier (Formula) | Final Multiplier | Rationale |
|-----------|--------------------|-----------|--------------------|-----------------|-----------|
| ecnl | ECNL Girls | 130 | 130/100 = 1.30 | **1.30×** | Girls Tier 1 co-#1 |
| ga | GA Girls | 140 | 140/100 = 1.40 | **1.40×** | Girls Tier 1 co-#1 |
| ga_aspire | GA ASPIRE Girls | 100 | 100/100 = 1.00 | **1.00×** | Post-fix calibration anchor |
| ecnl_rl | ECNL Regional League | 55 | 55/100 = 0.55 | **0.55×** | Tier 2 peer-validated |
| npl | National Premier League | 55 | 55/100 = 0.55 | **0.55×** | Tier 2 peer-validated |
| dpl | Development Players League | 55 | 55/100 = 0.55 | **0.55×** | Girls DPL peer to NPL |
| pre_ecnl | Pre-ECNL | 25 | 0.50× | **0.50×** | Tier 3; floor applies |
| local | Local/recreational | 20 | 0.50× | **0.50×** | Recreational; floor applies |
| unknown | Unclassified / NULL | 20 | 0.50× | **0.50×** | Conservative; flag for FORGE |

**Multiplier range:** 0.50× (local) to 1.50× (MLS NEXT). No tier exceeds 1.50× to prevent extreme rating volatility for teams with high-tier exposure.

**Note on undefined Tier 2 leagues** (Elite 64, EDP, NAL, USL Academy): These tiers share the Tier 2 designation with ECNL RL/NPL/DPL in the League Hierarchy but have no explicit calibration value. Assign multiplier = **0.80×** pending formal calibration — this places them between Tier 2 (0.55×) and the GA anchor (1.00×), conservative for currently unvalidated data. See `Rankings/Tier 2 Undefined Leagues — Calibration Recommendation.md` for the full recommendation.

---

## Section 3: Implementation Formula

### K-factor Application

```
K_effective = K_base × event_strength_multiplier
```

Where:
- `K_base` = the current base K-factor for the age group (as configured in `compute-rankings.js`)
- `event_strength_multiplier` = the value from Section 2 for the game's `event_tier`

### Application rules

| Design question | Phase 1 decision | Rationale |
|----------------|-----------------|-----------|
| Applied to both teams or one? | **Both teams symmetrically** | Same multiplier applies to both home and away team for a given game. A MLS NEXT game is high-signal for both teams, not just the winner. |
| Symmetric or asymmetric? | **Symmetric** | A game produces information about both teams equally. Asymmetric application would require per-team tier tracking within a single game — out of scope for Phase 1. |
| Applied at computation time or ingestion time? | **Computation time** | Multiplier is applied each time `compute-rankings.js` runs Phase 3. No new column needed in the games table. The `event_tier` column already exists; the multiplier is a lookup at runtime. |
| Schema change required? | **No schema change** | Multiplier is a runtime lookup from the CASE statement in `compute-rankings.js`. No new column, no migration. **FORGE dependency: none for schema.** |
| Code change required? | **Yes — `compute-rankings.js`** | The Phase 3 K-factor computation must be modified to apply the multiplier. ELO provides the CASE statement (Section 4); FORGE applies it. **FORGE dependency: code change to `compute-rankings.js` before execution.** |

### Formula applied in `compute-rankings.js` (Phase 3)

```javascript
// Phase 1 Event Strength Multiplier lookup
function getEventStrengthMultiplier(eventTier, gender) {
  const multipliers = {
    // Boys tiers
    mlsnext: 1.50,
    mlsnext_homegrown: 1.50,
    mlsnext_academy: 1.35,
    ecnl_boys: 1.20,
    ga_boys: 1.00,
    ga_aspire_boys: 1.00,
    ecnl_rl: 0.55,
    npl: 0.55,
    dpl: 0.55,
    pre_ecnl: 0.50,
    local: 0.50,
    // Girls tiers
    ecnl_girls: 1.30,
    ga_girls: 1.40,
    ga_aspire_girls: 1.00,
    // Tier 2 undefined (interim)
    elite64: 0.80,
    edp: 0.80,
    nal: 0.80,
    usl_academy: 0.80,
  };
  return multipliers[eventTier] ?? 0.50; // default: floor for unknowns
}

// In Phase 3 computation:
const multiplier = getEventStrengthMultiplier(game.event_tier, team.gender);
const K_effective = K_base * multiplier;
// Use K_effective in place of K_base for this game's Elo update
```

> **Note for FORGE:** The exact implementation depends on how `compute-rankings.js` currently handles K. If K is already modified in Phase 8 (League Calibration), coordinate so Phase 1 multipliers do not double-apply with existing calibration logic. ELO confirms: Phase 1 multiplier **replaces** any existing K-factor scaling from League Calibration for the same tier — it does not stack.

---

## Section 4: SQL Verification Package

> Phase 1 does not require SQL data changes (no schema change, no UPDATE to games table). The code change is in `compute-rankings.js`. The SQL below is for **verification** — run after FORGE applies the code change and after the Phase 1 recompute runs.

### V1 — Confirm event_tier Values Are Present (Pre-Execution Check)

Run before FORGE applies code change. Confirm all expected event_tier values exist in the DB:

```sql
-- Confirm all Phase 1 tiers have games in DB
SELECT
  event_tier,
  COUNT(*) AS game_count,
  COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) AS approx_teams
FROM games
WHERE is_hidden = false
  AND event_tier IS NOT NULL
GROUP BY event_tier
ORDER BY game_count DESC;

-- Expected: mlsnext, ecnl, ga, ga_aspire, ecnl_rl, npl, dpl, pre_ecnl all appear
-- Any NULL event_tier rows are flagged for FORGE to resolve (default multiplier 0.50× will apply)
```

### V2 — Rating Shift After Phase 1 Recompute (Post-Execution)

Run after Phase 1 recompute completes. Compare to pre-Phase-1 baseline (from April 29 Gate Results G4 snapshot):

```sql
-- Average rating by age_group and gender before vs. after Phase 1
-- Requires pre-Phase-1 ratings snapshot saved to a temp table or CSV before execution
-- Expected: ECNL/GA Girls ratings increase modestly vs. ECNL_RL/NPL/local teams
--           (higher-tier multiplier rewards each win more → ratings converge upward for better teams)

SELECT
  r.age_group,
  r.gender,
  ROUND(AVG(r.rating), 2) AS post_phase1_avg_rating,
  COUNT(*) AS team_count
FROM rankings r
WHERE r.rating IS NOT NULL
GROUP BY r.age_group, r.gender
ORDER BY r.gender, r.age_group;
-- Compare output against April 29 Gate Results baseline to compute delta
```

### V3 — Distribution Integrity Check (Post-Execution)

Confirm Phase 1 did not introduce outliers beyond the expected magnitude:

```sql
-- Count teams with rating change > 30 pts from April 29 baseline
-- Phase 1 should produce modest redistributive shifts, not large absolute swings
-- Alert threshold: > 200 teams with change > 30 pts is unexpected for Phase 1

-- [ELO note: exact query requires either a snapshot table or pre-Phase-1 CSV export.
--  Before executing Phase 1, Presten saves a CSV export of current rankings.
--  V3 then joins current ratings against the saved CSV.]

SELECT COUNT(*) AS large_movers
FROM rankings r
JOIN rankings_pre_phase1 p ON r.team_id = p.team_id
  AND r.age_group = p.age_group AND r.gender = p.gender
WHERE ABS(r.rating - p.rating) > 30
  AND r.rating IS NOT NULL;

-- PASS: large_movers <= 200
-- ALERT: large_movers > 200 → flag to SENTINEL before filing Phase 1 clearance
```

**Prerequisite for V3:** Before FORGE applies Phase 1 code change, Presten exports the current `rankings` table to a temporary table `rankings_pre_phase1`. One-line SQL:

```sql
CREATE TABLE rankings_pre_phase1 AS SELECT * FROM rankings;
-- Drop after Phase 1 validation is complete
```

---

## Section 5: Phase 1 Go/No-Go Pre-Execution Checklist

> Complete this checklist before any Phase 1 code changes or recompute. All items must be YES.

```
[ ] SENTINEL authorization received: YES / NO
    Briefing date of authorization: ___________

[ ] April 29 G1–G4 gate: CLEAN or CONDITIONAL (not BLOCKED)
    Gate result date: ___________

[ ] Calibration stability confirmed (per Post-Fix Calibration Stability Monitoring Spec)
    Stability confirmation date: ___________

[ ] Boys calibration scope decision made:
    [ ] Boys Option A PASS — Phase 1 applies to both Girls and Boys
    [ ] Boys Option A FAIL — Phase 1 applies Girls only; Boys K-multiplier remains 1.0× (baseline)
    Boys Option A result: ___________

[ ] FORGE has reviewed and confirmed Phase 1 code change to compute-rankings.js
    FORGE confirmation: ___________

[ ] Schema change required: NO (confirmed — no data column changes)

[ ] Pre-Phase-1 ratings snapshot saved to rankings_pre_phase1 table
    Snapshot saved at: ___________

[ ] V1 verification query run — all expected event_tier values present in DB
    V1 result: ___________
```

**If any item is NO or blank: do not execute Phase 1. File a queue item to SENTINEL.**

---

## Section 6: Post-Phase-1 Analysis Requirements

Within 48 hours of Phase 1 execution, ELO files a Phase 1 validation clearance in the next briefing using the following format:

```
Event Strength Phase 1 — ELO Validation Clearance
Execution date: ___________
Recompute completed: ___________

V1: PASS — [N] event_tier values confirmed present; [N] NULL-tier games default to 0.50×
V2: PASS / FLAG — rating delta for high-tier age groups: [direction, N pts]. [Consistent with / diverging from] Phase 1 rankings impact preview.
V3: PASS / ALERT — [N] large-mover teams (threshold: 200). [Within expected range / Investigate: ___________]

Phase 1 multiplier range confirmed: 0.50× – 1.50×. No tier outliers detected.

Event Strength Phase 1 validated. ELO recommends SENTINEL authorize Event Strength Phase 2 scoping session.
```

**Additional post-execution checks:**

1. **High-tier verification** — ECNL Girls and MLS NEXT Boys teams: do their ratings improve relative to GA_ASPIRE and ECNL_RL teams post-Phase-1? If the multiplier correctly rewards high-tier wins, the top-rated teams should increase slightly relative to mid-tier teams.

2. **Anomaly check** — Any teams with > 50-point shift that are not ECNL/GA/MLS NEXT dominant? Unexpected shifts may indicate a tier was miscategorized or the multiplier formula has an edge case.

3. **Boys isolation check (if Boys excluded)** — Confirm Boys ratings are unchanged post-Phase-1 recompute. Delta should be 0 ± 3 pts (recompute variance only).

---

*Spec filed 2026-04-27 by ELO. Execute only after SENTINEL authorization and all Section 5 pre-checks pass.*
