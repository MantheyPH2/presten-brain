---
title: Boys Option A — April 29 Execution Package
type: execution-package
author: ELO
date: 2026-04-29
status: ready
filed_at: "02:18"
g0_independent: true
related: "[[Boys Option A — Decision Document]]", "[[Boys Option A — April 29 Quick-Check Card]]", "[[Boys Option A Spot Check — Presten Execution Package]]", "[[Boys Option A — Fail Recovery Brief]]"
tags: [boys, calibration, club-rankings, dss, execution, option-a, april29, evo-draw]
---

# Boys Option A — April 29 Execution Package

> **G0 Independence confirmed.** This package runs regardless of G0 = GO or NO-GO. Boys Option A is a calibration-layer analysis — it does not depend on the GA ASPIRE fix or ecnl_verified backfill. Begin immediately when Presten opens a DB session on April 29.

---

## Section 1: What Boys Option A Is

Boys Option A is a 20–30 minute empirical spot check that determines whether Boys Club Rankings are accurate enough to include in the May 9 DSS demo. "Option A" refers to ELO's initial calibration design for Boys: no gender-separated calibration change — Boys use the same Glicko-2 parameters and league calibration values as Girls. The April 29 spot check tests whether that shared design produces credible Boys rankings in practice.

April 29 is the opening window because the Boys GA ASPIRE event reclassification (April 28 session) is now complete (or was deferred — either way, Boys GA ASPIRE carries cal=100, identical to Boys GA, so there is no net rating impact on Boys regardless). The Boys rankings in the database as of April 29 are the cleanest pre-DSS baseline available.

The verdict is **binary**: PASS (all three queries pass thresholds), FAIL (any query fails hard threshold), or YELLOW (WARN zone on Query 1 — requires ELO narrative judgment). PASS → Boys Club Rankings included at DSS. FAIL → Girls-Only Fallback activates immediately. YELLOW → ELO analysis within 48 hours before final decision.

If Option A PASSES, no calibration changes are made before DSS. If FAIL, Boys are excluded from Club Rankings at the demo and the post-DSS sprint Boys branch is deferred to May 15+.

---

## Section 2: Query Execution Package

Run queries in order. Save all output. Presten sends output to ELO for analysis (Section 3 filing).

**Estimated DB time: 20–30 minutes total.**

| Step | Query ID | Purpose | Required for Verdict? | Pass Threshold | FAIL Threshold | YELLOW Range |
|------|----------|---------|-----------------------|----------------|----------------|--------------|
| 1 | B-OA-1 | Boys vs. Girls avg rating divergence by age group | **REQUIRED** | All deltas ≤ 75 pts | Any delta > 100 pts | 76–100 pts (any age group) |
| 2 | B-OA-2 | Boys GA game volume — data sufficiency check | **REQUIRED** | All age groups ≥ 500 games | Any age group < 300 games | 300–499 games |
| 3 | B-OA-3 | Boys MLS NEXT rating distribution check | **REQUIRED** (if ≥10 MLS NEXT boys teams per age group) | Median 1,400–1,600; Top 10% ≥1,700; Bottom 10% ≤1,300 | Any metric outside range by >100 pts | Marginal deviation <100 pts |

---

### Query B-OA-1 — Boys vs. Girls Average Rating Divergence

```sql
SELECT
  r.age_group,
  r.gender,
  COUNT(*) AS team_count,
  ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating
FROM rankings r
WHERE r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND r.gender IN ('M', 'F')
  AND r.rank IS NOT NULL
GROUP BY r.age_group, r.gender
ORDER BY r.age_group, r.gender;
```

**Save output:**
```
\COPY (SELECT r.age_group, r.gender, COUNT(*) AS team_count, ROUND(AVG(r.rating)::numeric, 1) AS avg_rating, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating FROM rankings r WHERE r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17') AND r.gender IN ('M', 'F') AND r.rank IS NOT NULL GROUP BY r.age_group, r.gender ORDER BY r.age_group, r.gender) TO 'reports/boys-option-a-query1-2026-04-29.txt' WITH CSV HEADER
```

**Expected output:** 10 rows (5 age groups × 2 genders). ELO pivots to compute Boys − Girls delta per age group when filling the Decision Document.

**Record results in Decision Document Section 1 Query 1 table:**

| Age Group | Boys Avg | Girls Avg | Delta (Boys − Girls) | Result |
|-----------|----------|-----------|----------------------|--------|
| U13 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |
| U14 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |
| U15 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |
| U16 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |
| U17 | | | | PASS ≤75 / WARN 76–100 / FAIL >100 |

---

### Query B-OA-2 — Boys GA Game Volume Check

```sql
SELECT
  r.age_group,
  COUNT(DISTINCT g.id) AS boys_ga_game_count,
  COUNT(DISTINCT r.team_id) AS boys_team_count
FROM rankings r
JOIN games g ON (g.home_team_id = r.team_id OR g.away_team_id = r.team_id)
JOIN events e ON e.id = g.event_id
WHERE r.gender = 'M'
  AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND e.event_tier IN ('ga', 'ga_aspire')
  AND r.rank IS NOT NULL
GROUP BY r.age_group
ORDER BY r.age_group;
```

**Save output:**
```
\COPY (SELECT r.age_group, COUNT(DISTINCT g.id) AS boys_ga_game_count, COUNT(DISTINCT r.team_id) AS boys_team_count FROM rankings r JOIN games g ON (g.home_team_id = r.team_id OR g.away_team_id = r.team_id) JOIN events e ON e.id = g.event_id WHERE r.gender = 'M' AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17') AND e.event_tier IN ('ga', 'ga_aspire') AND r.rank IS NOT NULL GROUP BY r.age_group ORDER BY r.age_group) TO 'reports/boys-option-a-query2-2026-04-29.txt' WITH CSV HEADER
```

**Record results in Decision Document Section 1 Query 2 table.**

---

### Query B-OA-3 — Boys MLS NEXT Rating Distribution

```sql
SELECT
  r.age_group,
  COUNT(*) AS mlsnext_team_count,
  ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating,
  PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY r.rating) AS top_10pct_rating,
  PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY r.rating) AS bottom_10pct_rating,
  MAX(r.rating) AS max_rating
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id)
JOIN events e ON e.id = g.event_id
WHERE r.gender = 'M'
  AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND e.event_tier IN ('mls_next_homegrown', 'mls_next_academy', 'mls_next')
  AND r.rank IS NOT NULL
GROUP BY r.age_group
ORDER BY r.age_group;
```

**Save output:**
```
\COPY (...same query...) TO 'reports/boys-option-a-query3-2026-04-29.txt' WITH CSV HEADER
```

**Only assess age groups with ≥ 10 MLS NEXT teams.** Age groups below 10 teams: mark as "thin coverage — not evaluated."

**Record results in Decision Document Section 1 Query 3 table.**

---

## Section 3: Decision Criteria

ELO fills `Rankings/Boys Option A — Decision Document.md` with results and applies the following thresholds:

| Verdict | Conditions |
|---------|-----------|
| **Boys Option A: PASS** | All three of: (1) No age group in B-OA-1 shows delta >75 pts. (2) All U13B–U17B have ≥500 Boys GA games in B-OA-2. (3) MLS NEXT distribution (age groups with ≥10 teams) in expected range per B-OA-3. |
| **Boys Option A: YELLOW** | B-OA-1 shows 1–2 age groups in the 76–100 pt WARN zone (not >100). B-OA-2 shows one age group 300–499 games. B-OA-3 shows marginal deviation (<100 pts outside range). ELO writes a narrative assessment in Section 3 of Decision Document and proposes CONDITIONAL PROCEED to SENTINEL. |
| **Boys Option A: FAIL** | Any of: (1) Any age group in B-OA-1 delta >100 pts. (2) Any age group in B-OA-2 <300 Boys GA games. (3) Any MLS NEXT metric outside range by >100 pts (for age groups with ≥10 teams). |

**FAIL is the only automatic action trigger.** YELLOW and PASS both require ELO narrative in the Decision Document before SENTINEL authorization.

**If FAIL:** ELO files `Rankings/Boys Option A — Fail Recovery Brief.md` within 30 minutes. File is pre-written (dated 2026-04-28) — just confirm the trigger condition and file.

---

## Section 4: Post-Verdict Filing Protocol

| Verdict | ELO Action (within 30 min) | File Location | SENTINEL Notification |
|---------|---------------------------|--------------|----------------------|
| **PASS** | Fill Decision Document Sections 1–5. Mark task `task-2026-04-28-boys-option-a-fail-recovery-brief.md` as completed with note "Option A PASS — brief not needed." | `Rankings/Boys Option A — Decision Document.md` | Include in next ELO briefing. State: "Boys Option A PASS — all 3 queries clear thresholds. SENTINEL authorization requested." |
| **YELLOW** | Fill Decision Document Sections 1–5 including ELO narrative analysis. File SENTINEL queue item (priority: high) requesting CONDITIONAL PROCEED authorization. | `Rankings/Boys Option A — Decision Document.md` | File queue item immediately. Do not assume Boys are included until SENTINEL signs Decision Document. |
| **FAIL** | (1) File Fail Recovery Brief. (2) Update DSS Feature Readiness Tracker Boys cells. (3) File SENTINEL queue alert (priority: urgent). | `Rankings/Boys Option A — Fail Recovery Brief.md` | File queue alert immediately. State: "Boys Option A FAIL — Girls-Only Fallback activated. Boys Club Rankings excluded from DSS." |

**After any verdict:** Mark this task (`task-2026-04-28-boys-option-a-april29-execution-package.md`) `status: completed` with verdict noted.

---

## Section 5: May 1 Pipeline Interaction

**Boys Option A evaluation does NOT require delaying May 1 Stage 1 pipeline launch.** Boys Option A is a calibration-layer analysis (querying the existing rankings table). It does not modify the May 1 TYSA crawl configuration, the pipeline staging, or any FORGE infrastructure. These are fully independent workstreams.

During the Boys Option A evaluation window (April 29–May 5), the USL Academy Source #2 boys threshold is 50 pts (not standard 75 pts). This is a standing calibration parameter, not an Option A artifact. ELO notifies FORGE when the Option A verdict is final so FORGE can confirm this parameter remains appropriate post-verdict.

**G0 independence:** Boys Option A proceeds regardless of whether April 28 G0 = GO or NO-GO. The Boys ratings in the database are unaffected by the GA ASPIRE fix (Boys GA ASPIRE calibration = 100, same as Boys GA — no net rating impact). The G0 gate covers Girls calibration verification only.

---

## Quick Reference

```
Boys Option A — Decision Flowchart (April 29)

STEP 1: Run B-OA-1 (Boys/Girls divergence)
  ├── Any delta > 100 pts? → FAIL → File Fail Recovery Brief
  ├── Any delta 76–100 pts? → YELLOW (continue to B-OA-2)
  └── All deltas ≤ 75 pts? → PASS (continue to B-OA-2)

STEP 2: Run B-OA-2 (Boys GA game volume)
  ├── Any age group < 300 games? → FAIL (even if B-OA-1 PASS)
  ├── Any age group 300–499 games? → YELLOW
  └── All ≥ 500? → continue to B-OA-3

STEP 3: Run B-OA-3 (MLS NEXT distribution)
  ├── Any metric out of range >100 pts (≥10 teams/age group)? → FAIL
  ├── Marginal deviation? → YELLOW
  └── All within range? → PASS

FINAL VERDICT:
  Any FAIL at any step → Boys Option A: FAIL
  Any YELLOW, no FAIL → Boys Option A: YELLOW (ELO analysis)
  All PASS → Boys Option A: PASS → Boys in DSS
```

---

## References

- `Rankings/Boys Option A — Decision Document.md` — fill Sections 1–5 after queries run; SENTINEL authorization block
- `Rankings/Boys Option A — April 29 Quick-Check Card.md` — card format of the three queries (use this package instead for the full protocol)
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — original spot check spec with threshold derivation
- `Rankings/Boys Option A — Fail Recovery Brief.md` — file within 30 min of FAIL verdict (pre-written 2026-04-28)
- `Rankings/Boys Option A — Results Interpretation Guide.md` — ELO narrative guidance for Sections 3–4 of Decision Document
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration background
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — activates on FAIL
- `Rankings/Boys Option A — Verdict Filing Document.md` — SENTINEL-facing verdict summary

*ELO — 2026-04-29 02:18 | Filed before April 29 session open | G0-independent — proceed immediately*
