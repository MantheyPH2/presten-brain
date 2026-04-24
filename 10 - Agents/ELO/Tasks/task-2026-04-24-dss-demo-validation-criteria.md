---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: high
due: 2026-05-10
---

# Task: DSS Demo Validation Criteria — Pass/Fail Criteria for May 14 Data Validation Run

## Context

Presten's May 14 data validation run is scheduled. The DSS Demo Spec exists. What does not exist: explicit pass/fail criteria for the validation run.

Without criteria, "data validation" means Presten runs queries, things look roughly correct, and the go/no-go decision for the demo is a judgment call under time pressure two days before a high-stakes pitch. That is the wrong way to enter DSS.

With criteria, the validation run produces a checklist with boxes checked or unchecked. Either it passes or it doesn't. The demo is either safe or it isn't.

This task: write the explicit pass/fail criteria for the May 14 validation run. ELO is the right agent to write this because ELO knows what "correct" looks like from a calibration standpoint — what rating ranges are plausible for the anchor teams, what coverage numbers are defensible, and what band assignments should be expected.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/DSS Demo Validation Criteria — May 2026.md`

---

### Section 1: Anchor Team Requirements (MUST-PASS)

These teams must appear in the rankings for the demo to proceed. Define specific rank and rating thresholds based on ELO's calibration knowledge and the USA Rank comparison data (once the audit is run — use your best current estimate if the audit has not yet been executed, and note the source).

| Team | Age Group | Must Find | Max Acceptable Rank | Min Acceptable Rating |
|------|-----------|-----------|--------------------|-----------------------|
| Football Academy NJ (any name variation) | U13G | Yes | [ELO to fill] | [ELO to fill] |
| Wall Soccer Club Wall Elite Porto | U13G | Yes | [ELO to fill] | [ELO to fill] |
| FC Stars G2012 ECNL White | U13G | Preferred | [ELO to fill] | [ELO to fill] |
| NE Surf G2012 State Navy | U13G | Preferred | [ELO to fill] | [ELO to fill] |

Fill in the thresholds from your calibration analysis. Base your estimates on:
- Current USA Rank positions for these teams (from `Rankings/USA Rank Comparison 2026-04-23.md` — use the USA Rank reference positions as an anchor for expected relative placement)
- The Evo Draw rating distribution for U13G (from the threshold validation work)
- Your Brier score confidence interval (if the top 5 teams are within ±50 ranks of the held-out validation, that is acceptable)

**FAILURE CRITERION:** If Football Academy NJ is not findable in any name variation in U13G rankings, the demo is NOT safe for the rankings feature. SENTINEL escalation immediately.

Write the lookup query:
```sql
SELECT t.name, r.rating, r.rank
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%football academy%'
    OR t.name ILIKE '%FA NJ%'
    OR t.name ILIKE '%Football Academy NJ%')
ORDER BY r.rank;
```

---

### Section 2: Rank Band Assignment Requirements

The demo will show Rank Bands (Gold/Silver/Bronze). These assignments must be correct for the anchor teams:

| Team | Expected Band | Failure Assignment | Why |
|------|--------------|-------------------|-----|
| Football Academy NJ | Gold | Silver or Bronze | Demo anchor — must be Gold |
| Wall Soccer Club | [ELO to fill] | [ELO to fill] | Top-50 team — expected high |
| NE Surf G2012 State Navy | [ELO to fill] | Gold | Mid-field team should not be Gold |

Fill in expected bands from the Rank Bands Threshold Validation spec. Use the thresholds ELO pre-filled in that document.

Write the verification query using the actual threshold values (not placeholders):
```sql
SELECT t.name, r.rating, r.rank,
  CASE
    WHEN r.rating >= [GOLD_THRESHOLD] THEN 'Gold'
    WHEN r.rating >= [SILVER_THRESHOLD] THEN 'Silver'
    WHEN r.rating >= [BRONZE_THRESHOLD] THEN 'Bronze'
    ELSE 'Unranked'
  END AS band
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%football academy%'
    OR t.name ILIKE '%wall soccer%'
    OR t.name ILIKE '%NE Surf%')
ORDER BY r.rank;
```

**FAILURE CRITERION:** If Football Academy NJ is not Gold, Rank Bands are miscalibrated. This is a demo-blocking failure for that feature specifically.

---

### Section 3: Data Coverage Requirements

The demo claims comprehensive game coverage. Define the minimum counts that make the claim defensible:

**3.1 Total ranked teams in U13G:**
```sql
SELECT COUNT(*) FROM rankings 
WHERE age_group = 'U13' AND gender = 'F' AND rank IS NOT NULL;
```
Pass threshold: ≥ [ELO to fill] teams. Reasoning: ELO should specify the minimum that makes "comprehensive national rankings" a defensible claim versus a competitor like USA Rank. If USA Rank shows rankings for ~3,000 teams in U13G, Evo Draw should show at least [X]% of that to claim coverage parity.

**3.2 Teams with ≥ 5 games (reliable ratings):**
```sql
SELECT COUNT(DISTINCT t.id)
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND r.rank IS NOT NULL
  AND (SELECT COUNT(*) FROM games g 
       WHERE g.home_team_id = t.id OR g.away_team_id = t.id) >= 5;
```
Pass threshold: ≥ [ELO to fill] teams. Reasoning: teams with fewer than 5 games have high uncertainty — their ratings are noisy. Define the minimum percentage of ranked teams that should have reliable ratings.

**3.3 Total U13G games in the last 12 months:**
```sql
SELECT COUNT(*) FROM games
WHERE age_group = 'U13' AND gender = 'F'
  AND game_date >= NOW() - INTERVAL '12 months';
```
Pass threshold: ≥ [ELO to fill] games. Base this on what a credible coverage claim looks like — if Evo Draw claims to be comprehensive for national youth soccer, what's the floor for recent game volume in one age group?

For all three thresholds: document your reasoning (where the number comes from), not just the number.

---

### Section 4: Event Strength Requirements (Conditional)

**If Event Strength is implemented before May 14:**
1. At least one ECNL National event must have `strength_label = 'High'`
2. No ECNL National event should have `strength_label = 'Low'`
3. The U13G demo event identified in the Event Strength Post-Compute Validation spec must have `strength_label = 'High'`

Verification query:
```sql
SELECT event_name, strength_label, avg_team_rating
FROM event_strength
WHERE event_type ILIKE '%ECNL%' AND event_type ILIKE '%National%'
ORDER BY avg_team_rating DESC
LIMIT 10;
```

Pass: all ECNL National events labeled 'High' or 'Average'. Failure: any ECNL National event labeled 'Low'.

**If Event Strength is NOT implemented before May 14:**
Note this as a scope reduction, not a failure: "Event Strength feature is deferred. Demo will show rankings and Rank Bands only. Expected impact on DSS: [ELO's assessment of whether this weakens the demo significantly]."

---

### Section 5: May 14 Go/No-Go Decision Tree

Write the explicit decision:

```
MUST-PASS (any failure = demo not ready for that feature):
□ Football Academy NJ found in U13G rankings, rank ≤ [threshold]
□ Football Academy NJ labeled Gold
□ Total U13G ranked teams ≥ [threshold]

SHOULD-PASS (failure = flag to SENTINEL, but demo can proceed):
□ Wall SC found in top [threshold]
□ NE Surf found, labeled Silver or Bronze (not Gold)
□ Teams with ≥5 games count ≥ [threshold]
□ ECNL National events labeled High (if Event Strength implemented)

ACCEPTABLE GAPS (do not block demo):
□ Boys calibration not formally validated (Boys not in primary demo)
□ SnapSoccer ingestion not running
□ Club Rankings not implemented
□ USA Rank comparison table not populated

ESCALATION PROTOCOL:
- Any MUST-PASS failure → SENTINEL immediately, do not proceed with demo prep
- Multiple SHOULD-PASS failures (≥ 2) → SENTINEL, may need to scope-reduce demo features
- Single SHOULD-PASS failure → note it, monitor, prepare a fallback talking point
```

ELO fills in all [threshold] values.

---

### Section 6: Fallback Talking Points

For each SHOULD-PASS failure, write a one-sentence fallback talking point that Presten can use if the failure is noticed during the demo:

- Wall SC not in top 50: "Rankings reflect game-weighted performance — Wall's current position reflects their result in [most recent tournament], which may have affected their recency weight."
- Lower coverage count than expected: "We're actively adding game sources — our [X] pipeline integrations already capture more coverage than USA Rank for the Northeast."
- Event Strength not implemented: "Event Strength labeling is in our roadmap for this summer — today I'll walk you through the ranking engine itself."

Write ELO's version of these with specific numbers and calibration context.

---

## Definition of Done

- Anchor team rank and rating thresholds are specific numbers (not "should be near the top")
- Rank Band assignments for at least 3 anchor teams are specified using actual threshold values
- All 3 data coverage queries are written with specific pass thresholds and documented reasoning
- Event Strength requirements are written with IF/ELSE for implementation status
- May 14 decision tree has all threshold values filled in
- Fallback talking points are present for at least 2 SHOULD-PASS failure scenarios
- Summary in briefing: "DSS demo validation criteria filed. Football Academy NJ must appear at rank ≤ [X] and be labeled Gold. Total U13G ranked teams must exceed [Y]."

## Why This Matters

The May 14 validation run is the last safety check before DSS. Without explicit criteria, "data looks good" is the best outcome Presten can confirm. With criteria, the outcome is a checklist with pass/fail status on every demo-critical item. That's the difference between going into a pitch confident and going in hoping.

## References

- `Rankings/Rank Bands Threshold Validation — 2026-04-24.md` — Gold/Silver/Bronze threshold values for Section 2
- `Rankings/USA Rank Comparison 2026-04-23.md` — reference positions for anchor teams (Section 1)
- `Rankings/Event Strength Post-Compute Validation — 2026-04-24.md` — demo event selection (Section 4)
- `Product & Planning/DSS Demo Spec — May 2026.md` — demo structure and anchor team list
- Calibration Validation docs — for coverage count estimates in Section 3
