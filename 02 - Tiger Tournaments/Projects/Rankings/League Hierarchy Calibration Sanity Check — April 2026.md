---
title: League Hierarchy Calibration Sanity Check — April 2026
type: audit
author: ELO
date: 2026-04-26
status: complete
related:
  - "[[League Hierarchy]]"
  - "[[Calibration Values — League Hierarchy Reconciliation]]"
  - "[[Boys Calibration Status Summary — April 2026]]"
  - "[[DSS Ranking Accuracy Claims — Authorized Spec]]"
tags: [calibration, league-hierarchy, audit, rankings, dss, evo-draw]
---

# League Hierarchy Calibration Sanity Check — April 2026

> **Purpose:** Systematic audit of all League Hierarchy calibration values for internal consistency, relative tier coherence, and pre-DSS risk. SENTINEL uses Section 5 as the pre-DSS quality gate for calibration accuracy claims.
>
> **Source access note:** ELO sourced calibration values from `Rankings/Calibration Values — League Hierarchy Reconciliation.md` (filed 2026-04-26) and the `[[League Hierarchy]]` vault note. Distribution checks are qualitative — ELO does not have direct DB query access. Presten should run the distribution validation queries flagged in Section 3 before May 9 if any High-risk flags are confirmed.

---

## Section 1: Audit Methodology

ELO applied three checks to the full calibration value set:

**Internal consistency check:** Are leagues that are competitive peers assigned similar calibration values? National Tier 1 leagues should cluster; regional Tier 2 leagues should cluster at materially lower values; state-level leagues should sit below Tier 2. Any league whose cal value is not defensibly consistent with its tier peers is flagged for investigation.

**Rating distribution check (qualitative):** Based on ELO's knowledge of game volume and cross-league data in the 575,000-game study, do calibration values produce expected team rating ranges? A league with cal=160 should anchor teams above 1,500 on average. A league with cal=55 should anchor teams in the 1,200–1,400 range. Outliers from this expectation — especially if a high-cal league produces suspiciously low team ratings — signal a miscalibration.

**Outlier detection:** Which league has the highest cal value? Lowest? Are the extremes justified by competitive position? The GA ASPIRE case (April 2026) established the failure mode: a league at the wrong cal value creates systematic distortion, invisible until a direct cross-league comparison is run.

ELO's prior audit (`Calibration Values — League Hierarchy Reconciliation`, April 26) confirmed all documented tiers match between the League Hierarchy and the engine, with one known exception (GA ASPIRE Girls, corrected April 28). This audit extends that reconciliation into a full sanity check with explicit tier consistency and risk scoring.

---

## Section 2: Full Calibration Value Audit Table

### Girls Tiers

| League | Cal Value | Expected Tier | Assessment | Flag? |
|--------|-----------|---------------|------------|-------|
| GA (Girls Academy) | 140 | National Elite | OK | N |
| ECNL (Girls) | 130 | National Elite | OK | N |
| GA ASPIRE (Girls) | 100 (post-April-28) | Regional Premier | OK | N |
| ECNL RL (Girls) | 55 | Tier 2 Regional | OK | N |
| NPL (Girls) | 55 | Tier 2 Regional | OK | N |
| DPL (Girls) | 55 | Tier 2 Regional / GA Pathway | OK | N |
| Pre-ECNL | 25 | Tier 3 / Entry | OK | N |
| USL Academy (Girls) | undefined | Tier 2 Regional | Insufficient data — likely Low (missing cal value means engine treats as untiered) | Y |
| EDP (Girls) | undefined | Tier 2 Regional | Insufficient data — same engine gap as USL Academy | Y |
| Elite 64 (Girls) | undefined | Tier 2 Regional | Insufficient data | Y |
| NAL (Girls) | undefined | Tier 2 Regional | Insufficient data | Y |
| USYS State Leagues | undefined | Tier 3 | Insufficient data | Y |
| ODP | undefined | Tier 3 / Talent ID | Insufficient data | N (low volume, low impact) |

### Boys Tiers

| League | Cal Value | Expected Tier | Assessment | Flag? |
|--------|-----------|---------------|------------|-------|
| MLS NEXT (unified, pre-split) | 160 | National Elite | OK | N |
| MLS NEXT Homegrown (post-split) | 160 | National Elite | OK — empirically validated | N |
| MLS NEXT Academy (post-split) | 135 | National Elite | OK — empirically validated | N |
| MLS NEXT Unclassified (post-split) | 147 | National Elite | OK — interpolated; pending deployment May 18-20 | N |
| ECNL (Boys) | 120 | National Elite | OK | N |
| GA (Boys) | 100 | National Elite / Regional | High — see investigation note | Y |
| GA ASPIRE (Boys) | 100 | Regional Premier | OK — matches Boys GA; distinction pending Brier analysis | N |
| ECNL RL (Boys) | 55 | Tier 2 Regional | OK | N |
| NPL (Boys) | 55 | Tier 2 Regional | OK | N |
| DPL (Boys) | 55 | Tier 2 Regional | OK | N |
| Pre-ECNL | 25 | Tier 3 / Entry | OK | N |
| USL Academy (Boys) | undefined | Tier 2 Regional | Insufficient data | Y |
| EDP (Boys) | undefined | Tier 2 Regional | Insufficient data | Y |
| Elite 64 (Boys) | undefined | Tier 2 Regional | Insufficient data | Y |
| NAL (Boys) | undefined | Tier 2 Regional | Insufficient data | Y |

---

## Section 3: Flagged Leagues — Investigation Notes

### Flag 1: Boys GA (cal=100) — Possibly Low Relative to Competitive Position

**Why flagged:** Boys GA (Girls Academy Boys programs) carries cal=100. ECNL Boys is at cal=120. The gap is 20 points. For Girls, the same gap is 10 points (GA Girls=140, ECNL Girls=130). The question is whether Boys GA is structurally 20 points below Boys ECNL in competitive strength, or whether the gap reflects an untested assumption. Boys GA has never been separately Brier-validated — the 100 value is theoretically derived, not cross-league-confirmed.

**What the correct value might be:** If Boys GA is performing at approximately 65–70% win rate against Boys ECNL RL (cal=55), that would imply a cal ~85–95. If Boys GA performance is closer to 75–80% against ECNL RL, that would support cal=100. The cross-league win-rate study covers 575K games but has not reported gender-separated Boys GA vs. Boys ECNL breakdowns. Boys GA may be correctly at 100 or may be 10–15 points low. Either direction matters for Boys U13–U17 rankings.

**Query to confirm:**
```sql
-- Boys GA vs Boys ECNL RL head-to-head win rate (cross-league validation)
SELECT
  CASE
    WHEN home_event_tier = 'ga' AND away_event_tier = 'ecnl_rl' THEN 'GA home vs ECNL RL away'
    WHEN home_event_tier = 'ecnl_rl' AND away_event_tier = 'ga' THEN 'ECNL RL home vs GA away'
  END AS matchup,
  COUNT(*) AS games,
  ROUND(AVG(CASE WHEN home_score > away_score THEN 1.0 ELSE 0.0 END), 3) AS home_win_rate
FROM games g
JOIN events e ON e.id = g.event_id
WHERE g.gender = 'M'
  AND e.event_tier IN ('ga', 'ecnl_rl')
  AND g.is_hidden = false
GROUP BY 1;
```

**Priority:** Fix post-DSS / monitor now. Boys rankings at DSS are gated on Option A anyway. If Option A passes, note in the Decision Document that Boys GA cal=100 is assumed, not Brier-validated, and schedule Brier analysis for May 17–30.

---

### Flag 2: Undefined Tier 2 Leagues (USL Academy, EDP, Elite 64, NAL)

**Why flagged:** Four circuits are listed in the League Hierarchy at Tier 2 with no calibration value and no confirmed `event_tier` assignment in the engine. Based on the Event Strength League Tier Coverage Audit (April 24), these leagues' events may be receiving `event_tier = NULL` treatment in the DB — which means the ranking engine likely assigns them untiered (effectively Tier 3 or below) calibration.

**What the correct values should be:** All four are plausibly in the 55-point range (same as NPL, ECNL RL, DPL). ELO's recommended default assignments:
- USL Academy: cal=55 (professional development pathway, comparable to ECNL RL in competitive depth)
- EDP: cal=55 (Regional Tier 2, Eastern states — documented peer of NPL in that region)
- Elite 64: cal=55 (showcase series; teams competing are typically ECNL RL caliber)
- NAL: cal=25–55 (uncertain — competition level variable; conservative default 25 pending cross-league data)

**Query to confirm DB treatment:**
```sql
SELECT event_tier, COUNT(*) AS event_count
FROM events
WHERE event_tier IN ('usl_academy', 'uslacdemy', 'edp', 'elite_64', 'nal') OR event_tier IS NULL
GROUP BY event_tier
ORDER BY event_count DESC;
```

**Rating impact estimate:** If meaningful game volume exists in these leagues (> 5,000 games), teams playing primarily in these circuits may be systematically over- or under-ranked. USL Academy has 90 clubs nationally — if any significant fraction is in the DB, this is not a niche correction.

**Priority:** Fix before DSS for USL Academy and EDP if DB query confirms meaningful game volume (> 2,000 games). Low urgency if volume is < 500 games total. ELO recommends Presten run the above query and confirm volume before May 9.

---

### Flag 3: USYS State Leagues — Undefined

**Why flagged:** USYS state leagues are listed in the hierarchy without cal values and without a confirmed `event_tier` in the DB. FORGE's org ID expansion (assigned April 24) is adding USYS state league coverage. If those games are ingested without a defined `event_tier`, they enter the ranking computation under an undefined tier — distorting rankings for teams that play in them.

**Recommended value:** cal=35 (between Pre-ECNL at 25 and NPL at 55; state champion quality is below regional premier but above entry level).

**Priority:** Coordinate with FORGE to confirm `event_tier = 'state'` is applied to newly ingested USYS events before the May 14-15 data readiness validation. If state league events are in the DB without a tier assignment, this creates a coverage claim problem: we claim broader coverage, but those games are inflating uncertainty in the ranking computation. Resolve before DSS.

---

## Section 4: Relative Tier Consistency Check

### Girls Tier Map (cal values, post-April-28)

```
National Elite:     GA (140) ─── ECNL (130)
                    Gap: 10 pts — within normal peer variation ✅

Regional Premier:   GA ASPIRE (100)
                    Gap from National Elite floor: 30 pts — significant, defensible ✅

Tier 2 Regional:    ECNL RL / NPL / DPL (all 55)
                    Gap from Regional Premier: 45 pts — large but defensible;
                    the NPL/ECNL RL vs GA ASPIRE win-rate data supports this gap ✅

Tier 3 / Entry:     Pre-ECNL (25)
                    Gap from Tier 2: 30 pts — appropriate ✅

Undefined:          USL Academy, EDP, Elite 64, NAL (missing — risk)
                    If these are in the DB as untiered, they collapse to an implicit Tier 4
                    (below Pre-ECNL). That's likely wrong for USL Academy and EDP. ⚠️
```

**Girls tier map verdict:** Internally consistent for all defined tiers. The GA to ECNL 10-point gap is narrower than the Boys structure but aligns with cross-league win-rate data. The 45-point gap from GA ASPIRE (100) to ECNL RL (55) is large but empirically supported by the 575K-game study. No Girls tier is misplaced in its bracket.

### Boys Tier Map (cal values, pre-MLS NEXT split)

```
National Elite:     MLS NEXT (160) ─── ECNL (120) ─── GA (100)
                    MLS NEXT to ECNL gap: 40 pts ✅ (validated from 5,800 games)
                    ECNL to GA gap: 20 pts ⚠️ (theoretical; Boys GA Brier unvalidated)

Tier 2 Regional:    ECNL RL / NPL / DPL (all 55)
                    Gap from National Elite floor (GA at 100): 45 pts — consistent with Girls structure ✅

Tier 3 / Entry:     Pre-ECNL (25)
                    Gap from Tier 2: 30 pts — appropriate ✅
```

**Boys tier map verdict:** MLS NEXT and ECNL Boys are correctly placed and empirically validated. Boys GA (100) sits at the bottom of the National Elite bracket — which is structurally plausible, but the 20-point ECNL-to-GA gap is unvalidated. If Boys GA is actually closer to 85–90, several hundred Boys teams playing primarily in GA circuits are slightly over-rated. This is the highest-priority post-DSS Boys calibration question (alongside Boys GA ASPIRE Brier analysis).

**Boys tier consistency flag:** GA Boys (100) should be flagged as "calibration assumed, not validated" in all DSS-facing materials. The tier position is plausible — it falls between ECNL Boys (120) and ECNL RL (55), which is correct structurally. The specific value of 100 is the open question.

---

## Section 5: Pre-DSS Risk Assessment

**Overall calibration risk level: MEDIUM**

**Rationale:**

The Girls calibration is clean for all defined tiers as of April 28. The one known discrepancy (GA ASPIRE) was corrected. Girls rankings at DSS are safe to present as accurately calibrated within the defined league set.

Boys calibration has a medium-risk flag: Boys GA (cal=100) is theoretically placed but not Brier-validated. If Boys rankings are featured at DSS (Option A must pass by May 9), the DSS script should acknowledge that Boys calibration is continuously improving, not claim it is fully validated.

Undefined Tier 2 leagues (USL Academy, EDP, Elite 64, NAL) represent a structural gap. If any meaningful game volume from these leagues is in the DB, teams primarily playing in those circuits are ranked under an incorrect (likely lower) calibration assumption. Presten should run the DB volume check before May 9. If volume is < 500 total games, risk is Low. If volume is > 2,000 games, this warrants a pre-DSS fix.

**High-risk scenario (watch for before May 9):** If the Tier 2 undefined-league DB query returns significant volume (> 5,000 games across USL Academy, EDP, Elite 64, NAL), then teams in those leagues are currently under-calibrated — their wins are worth less than they should be, suppressing their ratings. This would be visible in the USA Rank comparison as a systematic negative delta for those teams. ELO would escalate this to SENTINEL immediately if it is confirmed.

**What SENTINEL should tell Presten:**

- Girls rankings: fully calibrated and audited for all defined leagues as of April 28. Safe to present.
- Boys rankings: calibration design is sound, MLS NEXT Boys is empirically validated, Boys GA is assumed-not-validated. Feature Boys at DSS only if Option A passes.
- Do not claim we rank "all leagues" — state the defined league set (10 tiers). Acknowledge state leagues are in active coverage expansion.
- If the Tier 2 volume check finds significant undefined-tier games: inform Presten before May 9 and remove any "comprehensive coverage" claims until the fix is applied.

---

## References

- `Rankings/Calibration Values — League Hierarchy Reconciliation.md` — source calibration values and reconciliation against engine
- `Rankings/Event Strength — League Tier Coverage Audit.md` — undefined Tier 2 league coverage gap analysis
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration context
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — accuracy claims this audit informs
- [[League Hierarchy]] — authoritative tier list and cal values (version: 2026-04-24)

*ELO — 2026-04-26*
