---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Implementation Spec.md"
---

# Task: Event Strength Phase 1 — Implementation Spec

## Context

Event Strength Phase 1 authorization is contingent on April 28 completing successfully and calibration stability being confirmed by May 5. SENTINEL has pre-defined the authorization gate: if G1–G4 pass on April 29 and calibration stability is confirmed, Phase 1 is authorized. The Phase 1 authorization criteria document exists. The Phase 1 data coverage check exists. The Phase 1 rankings impact preview exists.

What does not exist: the actual implementation spec. ELO has documented extensively what Phase 1 will produce and under what conditions it will run — but has not written what FORGE and ELO actually do in the Phase 1 execution session. When SENTINEL authorizes Phase 1 (potentially May 5), ELO's next session begins with: "what is the first command?"

This task closes that gap. Writing the implementation spec before authorization means Phase 1 execution begins immediately after SENTINEL's authorization signal — no design session, no ambiguity about execution order.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Implementation Spec.md`

---

### Section 1: Phase 1 Scope

Define exactly what Phase 1 covers. Based on the authorization criteria and data coverage check:

**In scope for Phase 1:**
- All currently active leagues with calibrated event_tier values in the League Hierarchy
- Event strength as a multiplier on the K-factor for each game (not a separate score — applied at rating computation time)
- Girls age groups only if Boys calibration is not yet confirmed (per Club Rankings conditional scope pattern)

**Out of scope for Phase 1:**
- New event_tier classifications not yet in the League Hierarchy
- SnapSoccer tier (not yet ingested)
- Manual event strength overrides
- Phase 2 features (state-level weighting, tournament prestige adjustments, etc.)

ELO states the scope explicitly. SENTINEL uses this to answer Presten's question: "What does Event Strength Phase 1 actually do that Phase 0 doesn't?"

---

### Section 2: League Tier → Strength Value Mapping

The core Phase 1 data: what K-factor multiplier does each event_tier receive?

ELO produces this table from the League Hierarchy. For every event_tier value currently in the DB:

| event_tier | League / Event Type | Phase 0 K-factor (baseline = 1.0×) | Phase 1 Strength Multiplier | Rationale |
|-----------|--------------------|------------------------------------|----------------------------|-----------|
| mlsnext | MLS NEXT | 1.0 | [ELO fills] | |
| ecnl | ECNL | 1.0 | [ELO fills] | |
| ecnl_rl | ECNL Regional League | 1.0 | [ELO fills] | |
| ga | GA Soccer | 1.0 | [ELO fills] | |
| ga_aspire | GA ASPIRE (post-fix) | 1.0 | [ELO fills] | |
| npl | NPL | 1.0 | [ELO fills] | |
| dpl | DPL | 1.0 | [ELO fills] | |
| usys | USYS State Cup / Regional | 1.0 | [ELO fills] | |
| local | Local/recreational | 1.0 | [ELO fills] | |
| [others from DB] | | 1.0 | [ELO fills] | |

**Design constraint:** Multiplier range must be bounded. ELO states the min/max multiplier it will use and why. Example: "Multipliers range from 0.5× (local) to 1.5× (ECNL/MLS NEXT). No tier exceeds 1.5× to prevent extreme rating volatility for teams with high-tier exposure."

ELO confirms these values with the League Hierarchy calibration values. Any tier with `calibration_value` in the League Hierarchy should derive its multiplier from that value according to a stated formula.

---

### Section 3: Implementation Formula

How does ELO apply the event strength multiplier to the existing rating computation?

Specify:
- Where in the ELO formula the multiplier applies: `K_effective = K_base × event_strength_multiplier`
- Whether the multiplier applies to both teams or only the expected-underdog team
- Whether the multiplier is symmetric (same K for both teams in a game) or asymmetric
- Whether the multiplier applies at rating computation time (each game reprocessed with new K) or at event ingestion time (stored with the game record)

If the implementation requires a schema change (e.g., adding an `event_strength` column to the games table), state this. FORGE must know about schema changes before they execute. Do not introduce a schema change without flagging it as a FORGE dependency.

---

### Section 4: SQL Update Package

Write the SQL that implements Phase 1.

**Step 1 — Populate event_strength values**

If event strength is stored per game:
```sql
-- Update games table with event_strength multiplier based on event_tier
-- ELO writes the actual UPDATE statement
UPDATE games
SET event_strength = CASE
  WHEN event_tier = 'mlsnext' THEN [multiplier]
  WHEN event_tier = 'ecnl' THEN [multiplier]
  -- ... all tiers
  ELSE 1.0
END
WHERE event_strength IS NULL;
-- Expected: [N] rows updated (all games with event_tier values)
```

If event strength is computed at ranking time (no column change):
- State the formula change in `compute-rankings.js` or equivalent
- FORGE must apply the code change. ELO files the formula; FORGE implements it.

**Step 2 — Recompute rankings with Phase 1 K-factors**
```sql
-- Trigger or run command:
-- [ELO specifies the correct command — same as the April 28 recompute, but with Phase 1 K-factors active]
```

**Step 3 — Verification queries**

Three queries to run immediately after Phase 1 recompute:

```sql
-- V1: Confirm event_strength values are populated
-- V2: Check that high-tier teams (ECNL, MLS NEXT) show higher average K-effective than low-tier teams
-- V3: Confirm rating distribution has not dramatically shifted (compare avg ratings before vs. after Phase 1)
```

ELO writes all SQL.

---

### Section 5: Phase 1 Go/No-Go Criteria

Before executing the Phase 1 UPDATE (Step 4-1), ELO runs a pre-execution check:

```
[ ] Calibration stability confirmed (per Post-Fix Calibration Stability Monitoring Spec): YES
[ ] event_tier values in DB match the tier mapping table (Section 2): verified
[ ] Schema change required: YES / NO → if YES, FORGE has been assigned and confirmed schema change
[ ] Pre-Phase-1 ratings snapshot saved (baseline for post-Phase-1 comparison): saved at [path]
[ ] SENTINEL authorization received: YES (briefing date: ___)
```

If any pre-check is NO: do not execute Phase 1. File a queue item to SENTINEL.

---

### Section 6: Post-Phase-1 Analysis Requirements

After Phase 1 executes, ELO runs within 48 hours:

1. **Rating shift analysis** — compare pre-Phase-1 vs. post-Phase-1 ratings for a sample of teams across all event tiers. Does the direction and magnitude match the Phase 1 rankings impact preview?
2. **High-tier verification** — ECNL and MLS NEXT team ratings: did they improve relative to GA teams as expected?
3. **Anomaly check** — any teams with > 50-point shift that weren't in GA ASPIRE affected groups? Unexpected shifts may indicate multiplier misconfiguration.

ELO files results in the next briefing after Phase 1 execution.

---

## Definition of Done

- Spec written at `Rankings/Event Strength Phase 1 — Implementation Spec.md`
- All 6 sections present
- Tier-to-multiplier mapping table (Section 2) is fully populated — no blank cells
- Implementation formula (Section 3) is specific enough that FORGE can implement it without asking ELO for clarification
- SQL update package (Section 4) contains actual SQL — not pseudocode
- Any required FORGE dependencies (schema changes, compute-rankings.js changes) are explicitly flagged in Section 3
- Summary in briefing: "Phase 1 implementation spec written. Tier mapping: [N] tiers. Multiplier range: [min]×–[max]×. Schema change required: YES/NO. FORGE dependency: YES/NO. Ready to execute on SENTINEL authorization."

## Why This Matters

When SENTINEL says "Phase 1 authorized," ELO's response time determines when the ranking improvement is live. Without a spec, ELO spends the first authorized session writing the implementation. With this spec ready, ELO's first authorized session is execution — Phase 1 can be live the same day SENTINEL authorizes. For the DSS demo timeline, a 1-day execution cycle vs. a 3-day design+execution cycle may be the difference between Event Strength being in the DSS demo or not.

## References

- `Rankings/Event Strength Phase 1 — Authorization Criteria.md` — authorization conditions
- `Rankings/Event Strength Phase 1 — Data Coverage Check.md` — confirms which tiers have sufficient data
- `Rankings/Event Strength Phase 1 — Rankings Impact Preview.md` — predicted outcomes (spec must match)
- `Rankings/League Hierarchy.md` — source of truth for event_tier calibration values
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability pre-check gate
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Phase 1 as DSS feature
