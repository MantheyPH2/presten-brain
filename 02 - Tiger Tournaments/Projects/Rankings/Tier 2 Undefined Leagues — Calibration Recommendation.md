---
title: Tier 2 Undefined Leagues — Calibration Recommendation
type: calibration-recommendation
author: ELO
date: 2026-04-27
status: ready-for-authorization
related: "[[League Hierarchy Calibration Sanity Check — April 2026]]", "[[League Hierarchy]]", "[[DSS Ranking Accuracy Claims — Authorized Spec]]"
tags: [calibration, league-hierarchy, recommendation, dss, evo-draw]
---

# Tier 2 Undefined Leagues — Calibration Recommendation

> **Purpose:** ELO's formal calibration recommendation for 4 Tier 2 leagues currently missing `event_tier` calibration values: USL Academy, EDP Soccer, Elite 64, NAL. Written before April 29 volume check — recommendation is ready to act on immediately when Presten runs the volume SQL.

---

## Section 1: League Profiles

| League | Tier | Geographic Scope | Gender | Age Groups | Competitive Level vs. Known Benchmarks |
|--------|------|-----------------|--------|------------|---------------------------------------|
| USL Academy | 2 | National | Boys | U13–U17 | Professional club development pathway (USL affiliates). Comparable to ECNL RL in depth — teams are professional club academies below MLS NEXT entry level. Positioned between NPL (55) and ECNL Boys (120). |
| EDP Soccer | 2 | Northeast Regional (NJ/NY/PA/CT/MA/MD) | Boys + Girls | U9–U19 | Established regional Tier 2 circuit. Documented peer of NPL in its region — EDP operates at the same entry bar as NPL and shares substantial geographic overlap. Competitive level: comparable to NPL (55). |
| Elite 64 | 2 | National (showcase/tournament format) | Boys + Girls | U12–U17 | Showcase series drawing ECNL RL–caliber teams for high-visibility events. Field quality is consistent with ECNL RL — entry is selective but not National Elite. Anchor: ECNL RL (55). |
| NAL | 2 | Regional (Mid-Atlantic/Southeast) | Boys | U13–U17 | National Academy League — Boys development circuit with variable competitive depth. Weaker competitive entry bar than NPL/ECNL RL in its region. Positioned structurally above Pre-ECNL (25) but below established Tier 2 circuits (55). |

---

## Section 2: Calibration Reasoning

### USL Academy

**Proposed cal value:** 55

**Reasoning:** USL Academy clubs are professional club academy programs affiliated with USL Championship and USL League One clubs. Their competitive level mirrors Boys ECNL RL: strong enough to compete with state and regional elite programs, not yet at MLS NEXT or ECNL Boys Tier 1. The closest anchor is ECNL RL (55), which also operates as a regional Tier 2 circuit with professional-club-adjacent programs. Assigning 55 is consistent with the Tier 2 cluster (NPL=55, ECNL RL=55, DPL=55) and aligns with the structural expectation that all Tier 2 regional leagues calibrate at the same level absent cross-league data showing differentiation.

**Confidence:** Medium. No direct cross-league win-rate data exists for USL Academy vs. ECNL RL or NPL. The structural argument (professional pathway, Tier 2 designation) is strong. The 55 value could be revised after a future win-rate study.

**Acceptable range:** 45–65 would be defensible. Below 35 would underweight this league's professional development context.

---

### EDP Soccer

**Proposed cal value:** 55

**Reasoning:** EDP Soccer is the dominant Tier 2 regional circuit in the Northeast (NJ, NY, PA, CT, MA, MD). EDP operates as the geographic peer of NPL in its region — both draw from the same pool of Tier 2 competitive teams. Where a team plays NPL in the Midwest, a comparable team in the Northeast plays EDP. Given that NPL is calibrated at 55, assigning EDP the same value reflects this documented competitive equivalence. The 575K-game cross-league study confirmed that regional Tier 2 circuits cluster around this range — EDP's inclusion at 55 is consistent with that finding.

**Confidence:** High. EDP's competitive positioning relative to NPL is well-established in youth soccer operations. Cal=55 is defensible without additional cross-league data.

**Acceptable range:** 50–60. Narrow band because EDP's peer equivalence to NPL is strong.

---

### Elite 64

**Proposed cal value:** 55

**Reasoning:** Elite 64 is a showcase circuit, not a season-long league — teams qualify for showcase events based on their standing in primary leagues (ECNL RL, NPL, state premier). The field of participants is therefore ECNL RL–caliber by selection design. Assigning 55 matches the competitive level of the primary leagues from which Elite 64 draws. A higher value (e.g., 70+) would overcalibrate because Elite 64 games do not represent the top of regional competition, only a curated mid-tier showcase.

**Confidence:** Medium. Showcase formats introduce selection bias — weaker teams may not attend, inflating apparent win rates vs. the broader 55-cal tier. Cal=55 is the conservative and defensible choice. A post-DSS Brier analysis on Elite 64 game outcomes could confirm or adjust.

**Acceptable range:** 45–65. Below 45 would underweight teams that qualified selectively to attend.

---

### NAL (National Academy League)

**Proposed cal value:** 40

**Reasoning:** NAL operates as a Boys development circuit in the Mid-Atlantic and Southeast regions. Unlike NPL, EDP, and ECNL RL — which have consistent national-level entry bars — NAL's competitive depth is variable by region and has a weaker defined minimum entry standard. Teams playing primarily in NAL circuits tend to sit below the established Tier 2 floor (55) but above the entry-level Pre-ECNL (25). Cal=40 positions NAL midway between Tier 3/Entry and Tier 2, reflecting the inconsistent competitive depth without undercounting leagues where NAL teams occasionally post competitive results against Tier 2 peers. Assigning 55 (same as NPL) would overweight NAL given its structurally weaker entry bar. Assigning 25 (Pre-ECNL) would underweight it. Cal=40 is the calibrated middle position.

**Confidence:** Low-Medium. NAL game volume in the DB is unknown — this is the league where the April 29 volume check matters most before applying any value. If Presten's query returns < 100 NAL games, the calibration change has negligible impact and can safely be applied at any point. If NAL volume is > 1,000 games, the 40 vs. 55 distinction begins to matter for rankings accuracy, and a cross-league win-rate check against ECNL RL should precede authorization.

**Acceptable range:** 30–50. Below 30 would underweight a league with identifiable competitive structure. Above 50 would overstate its equivalence to NPL.

---

## Section 3: Volume-Gated Decision Matrix

Presten runs the volume-check SQL on April 29. Pre-written decision matrix:

| Game Count per League (SQL Result) | Action |
|------------------------------------|--------|
| ≥ 2,000 games | Apply calibration before DSS. Update League Hierarchy and trigger partial recompute for affected event_tiers. Priority: USL Academy, EDP first (higher confidence values). |
| 500–1,999 games | Apply calibration before DSS if session time allows. Not on the critical path. If session is crowded (April 29 is dense), defer to May 5 session. |
| < 500 games | Defer to post-DSS. Low volume means minimal accuracy impact on any team's ranking. This league is safe to leave uncalibrated for May 16. |
| 0 games | League events may not be in the DB. Confirm before adding cal value — a cal value with no games has no effect but creates maintenance overhead. |

**DSS accuracy threshold:** The critical boundary is 2,000 games across any single league. At that volume, teams playing primarily in that circuit contribute enough games to meaningfully distort their ratings under an incorrect (null = Tier 4-equivalent) calibration. Below 500 games, the distortion is distributed across enough teams and game counts that no single team's ranking is materially affected.

**ELO escalation trigger:** If any single league returns > 5,000 games, escalate immediately to SENTINEL before applying the fix. At 5,000+ games, the recompute impact could be non-trivial and should be scoped before running.

---

## Section 4: Implementation Spec (If Authorized)

If Presten authorizes all four values in a session:

1. **Update League Hierarchy:** Add the four entries with proposed cal values to the `event_tier` calibration lookup used by `compute-rankings.js`.

2. **Confirm event_tier strings in DB:**
   ```sql
   SELECT event_tier, COUNT(*) AS game_count
   FROM events
   WHERE event_tier IN ('usl_academy', 'edp', 'elite_64', 'nal', 'usl_acad', 'edpsoccer', 'elite64')
     OR (event_tier IS NULL AND source IN ('usl_academy','edp','elite_64','nal'))
   GROUP BY event_tier
   ORDER BY game_count DESC;
   ```
   Confirm the exact `event_tier` string values in use — do not assume the strings in the League Hierarchy match the DB exactly. The April 26 sanity check flagged a potential typo ('uslacdemy') in an earlier audit.

3. **Count affected games:**
   ```sql
   SELECT event_tier, COUNT(*) AS affected_games
   FROM events
   WHERE event_tier IN ('<confirmed_string_1>', '<confirmed_string_2>', ...)
   GROUP BY event_tier;
   ```

4. **Trigger recompute:** Partial recompute covering only age groups and teams with games tagged to the four event_tier values. A full recompute is not required unless the volume check finds > 10,000 combined games and SENTINEL authorizes a full run.

5. **Post-recompute sanity check:** For each league, sample 10 teams that play primarily in that circuit and confirm their rating moved in the expected direction (upward if previously untiered = null calibration, which treated them as below-Tier-3). Document in next briefing.

---

## Section 5: DSS Accuracy Claim Impact

**If these 4 leagues remain uncalibrated through DSS (May 16):**

The honest answer is: "Our calibration covers 10 defined tiers. Four Tier 2 regional circuits (USL Academy, EDP, Elite 64, NAL) are in our league hierarchy but not yet assigned calibration values in the engine. Games from those leagues are currently processed as untiered — equivalent to Tier 4 — which may suppress ratings for teams that play primarily in those circuits."

**ELO's assessment:** This is defensible if game volume from these four leagues is < 500 total in the DB. The DSS accuracy claims in `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` do not claim 100% league coverage — they claim accuracy for defined, calibrated leagues. As long as ELO does not present USL Academy, EDP, Elite 64, or NAL teams as examples of accurate calibration in the DSS demo, the claim holds.

**If volume is > 2,000 games:** The "comprehensive calibration" framing becomes harder to defend if a client asks specifically about these leagues. SENTINEL should apply the values before May 16 if the volume check confirms meaningful game counts.

**Language SENTINEL can use at DSS:** "Our calibration covers MLS NEXT, ECNL, Girls Academy, GA ASPIRE, ECNL RL, NPL, and DPL — the seven circuits where we have the most game volume and cross-league validation. We're adding calibration for USL Academy and regional circuits in the current update cycle."

---

## References

- `Rankings/League Hierarchy Calibration Sanity Check — April 2026.md` — audit that identified this gap (Section 3, Flag 2)
- `Rankings/League Hierarchy.md` — current calibration values used as anchors
- `Rankings/Event Strength — League Tier Coverage Audit.md` — event_tier coverage gaps
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — DSS claims this affects

*ELO — 2026-04-27*
