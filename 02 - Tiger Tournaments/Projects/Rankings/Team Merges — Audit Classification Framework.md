---
title: Team Merges — Audit Classification Framework
type: classification-framework
author: ELO
date: 2026-04-26
status: ready
related: "[[Team Merges Audit — Presten Execution Package]]", "[[Merge Audit Triage Framework — May 2026]]", "[[Pre-DSS Team Merge Audit — U13G]]"
tags: [team-merges, data-quality, dss, classification, evo-draw]
---

# Team Merges — Audit Classification Framework

> **Purpose:** Define how ELO classifies merge rows when Presten's audit results arrive. Every merge row receives exactly one Type classification via the decision tree in Section 3. Classifications are deterministic — an observer other than ELO applying the same tree to the same row should reach the same result.
>
> **When this activates:** Presten runs `Rankings/Team Merges Audit — Presten Execution Package.md` queries (window: April 29 – May 5). ELO receives output files and applies this framework. Classification begins immediately — do not wait for all batches.

---

## Section 1: Merge Type Taxonomy

Four types cover all expected merge cases in the 27,820 `team_merges` entries.

---

**Type A — Same Team, Same Club, Name Variant**

Definition: Two records representing the same team, same club, same age group — only the name format differs. Examples:
- `"FC Dallas U15 Girls"` merged with `"FC Dallas Girls U15"`
- `"PDA Blue U14G"` merged with `"PDA U14 Girls Blue"`
- `"Football Academy NJ"` merged with `"Football Academy New Jersey"`

Action: **CONFIRM MERGE.** This is the correct and intended use of `team_merges`. No rating contamination.

Confidence required: High — ELO confirms same `club_id`, same age group, same gender, name is clearly a variant (abbreviation, word reorder, punctuation difference).

---

**Type B — Same Club, Plausible Merge, Needs Verification**

Definition: Two records from the same club with overlapping age group and gender, but the name relationship is not obvious. Could be a correct merge (club renamed a team, or club folded two squads) or an incorrect merge (two distinct teams at the same club in the same age group).

Examples:
- Club has `"U14G Blue"` and `"U14G Red"` — two squads that should NOT be merged
- Club has `"U14G 2024"` and `"U14G 2025"` — could be year-cohort variants of the same team, or genuinely separate squads

Action: **VERIFY BEFORE CONFIRMING.** ELO flags for Presten review with a specific yes/no question: "Are [Team A] and [Team B] the same team or different squads at [Club]?"

Do not classify as Type A without confirmation. Do not classify as Type C/D unless club or age group divergence is confirmed.

---

**Type C — Cross-Club Merge (Likely Error)**

Definition: Two records with different `club_id` values merged together. This combines rating histories across clubs — a team's ELO history now includes games played by a different club's team.

Examples:
- `club_id = 441` (FC Dallas) merged with `club_id = 992` (FC Frisco) — completely different organizations
- Could appear in data as a name collision: two clubs both have a team named `"U15G Premier"`

Action: **FLAG AS ERROR.** Cross-club merges almost certainly inflate or deflate ratings by contaminating rating histories with games from a different club.

Severity: HIGH — fix required before ratings are used in DSS demo or club rankings.

---

**Type D — Same Club, Different Age Group Merge (Likely Error)**

Definition: Same `club_id` but different age groups merged — e.g., a `U14G` record merged with a `U15G` record.

Examples:
- Club has `"U14/15G Premier"` and data incorrectly assigned it to both U14 and U15 age groups
- A team aged up mid-season and games from both age groups ended up under one merged record

Action: **FLAG AS ERROR.** Different age groups have different calibration values (K-factor, RD floor) and should never share a rating history. Age group divergence in a merge is always incorrect.

Severity: HIGH — fix required.

---

## Section 2: Severity Levels

| Severity | Definition | SENTINEL Action Required? |
|----------|------------|--------------------------|
| CONFIRMED | Type A — verified correct name variant, same club/age group/gender | No |
| VERIFY | Type B — ambiguous, Presten confirmation needed | Presten email or session clarification requested |
| HIGH — FIX | Type C or D — cross-club or cross-age-group error, rating contamination | Yes — SENTINEL flags for correction; FORGE owns fix |
| HIGH — INVESTIGATE | Cannot classify without additional data (e.g., `club_id` lookup failed, age group inconsistent in DB) | Yes — ELO describes exactly what data is needed |

---

## Section 3: ELO Classification Decision Tree

Apply steps in order to every merge row:

```
Step 1: Do the two merged records share the same club_id?
  NO  → Type C (Cross-Club Error)  → Severity: HIGH-FIX → STOP
  YES → continue to Step 2

Step 2: Do the two merged records share the same age group AND gender?
  NO  → Type D (Age Group or Gender Error) → Severity: HIGH-FIX → STOP
  YES → continue to Step 3

Step 3: Is the name relationship between the two records clearly a variant?
  (Variants: abbreviation, word order, punctuation, spelling,
   year suffix, color suffix where only one color exists for that club/age group)
  YES → Type A (Confirmed Name Variant) → Severity: CONFIRMED → STOP
  UNCERTAIN → continue to Step 4

Step 4: Do the two records share overlapping game history?
  (Check: same event IDs, same opponent IDs within the same season)
  YES → Type A (Confirmed by Game Overlap) → Severity: CONFIRMED → STOP
  NO or UNKNOWN → Type B (Needs Verification) → Severity: VERIFY → flag for Presten
```

ELO does not skip steps. Every row starts at Step 1. If `club_id` is NULL in both records, ELO classifies as HIGH — INVESTIGATE and notes the DB gap.

---

## Section 4: Output Format

ELO files all classification results in this table format. File in batches as results arrive — do not wait until all rows are reviewed.

| merge_id | team_1_name | team_2_name | club_id_1 | club_id_2 | age_group | gender | Type | Severity | ELO Determination | Action Required |
|----------|-------------|-------------|-----------|-----------|-----------|--------|------|----------|-------------------|-----------------|
| | | | | | | | | | | |

**Batch file naming:** `Reports/team-merges-classifications-batch-[N]-[date].md`

**Filing cadence:** File each batch within one session of receiving results from Presten. Do not accumulate batches. SENTINEL reviews each batch file as it arrives.

---

## Section 5: Rating Impact Assessment Protocol

For every Type C or Type D row, ELO performs a quick rating impact assessment before filing the batch. This is filed as a one-paragraph note appended to the row in Section 4.

**Assessment steps:**

1. **Identify affected teams.** Which team IDs are involved in the incorrect merge? Are either team in the top 500 rankings for their age group?

2. **Estimate contamination period.** How many games are in the shared rating history since the merge was created? Pull from the game count in Q1 output if available.

3. **Severity of impact:**
   - ≥ 20 games contaminated → HIGH rating impact. Likely visible in current ratings.
   - < 20 games contaminated → MEDIUM rating impact. May not be visible at current rating levels.
   - 0 games since merge (new/inactive teams) → LOW. Fix is preventive.

4. **Fix required:** State what correction is needed and who owns it.
   - Type C fix: Split the merge record. Update `primary_team_id` in `team_merges`. Full recompute required for both affected teams. Owner: FORGE.
   - Type D fix: Same as Type C, plus confirm correct age group assignment. Owner: FORGE.
   - If a demo team (Football Academy NJ, PA Classics, PDA) is involved: flag to SENTINEL immediately regardless of severity level.

---

## Section 6: Summary Report Template

ELO files the summary report when 80% or more of Presten's provided rows are classified, or by May 5, whichever comes first.

```
Team Merges Audit — Classification Summary
Date: [date]
Rows reviewed: [N] of [total provided by Presten]
Batches from Presten: [N]
Batches filed by ELO: [N]

Classification breakdown:
  Type A (CONFIRMED):            [N] — [%] of reviewed
  Type B (VERIFY):               [N] — [%] of reviewed
  Type C (Cross-Club Error):     [N] — [%] of reviewed
  Type D (Age Group Error):      [N] — [%] of reviewed
  HIGH — INVESTIGATE:            [N] — [%] of reviewed

HIGH-FIX items: [N total]
  - [List each: merge_id, team names, type, estimated games contaminated]

VERIFY items pending Presten response: [N]
  - [List top 5 by potential rating impact with specific question for Presten]

Demo team status:
  Football Academy NJ:  CONFIRMED / FLAGGED / NOT IN AUDIT BATCH
  PA Classics:          CONFIRMED / FLAGGED / NOT IN AUDIT BATCH
  PDA:                  CONFIRMED / FLAGGED / NOT IN AUDIT BATCH

DSS Feature Readiness Tracker update:
  Rank Bands demo team merge status: CONFIRMED / FLAGGED / PENDING
  Club Rankings primary clubs merge status: CONFIRMED / FLAGGED / PENDING

ELO confidence in unreviewed rows (27,820 − [N reviewed]):
  Confidence level: HIGH / MEDIUM / LOW
  Reason: [one sentence — e.g., "HIGH-FIX rate is [X]% in reviewed sample; extrapolating, estimated [N] errors remain unreviewed"]

Recommended next step for SENTINEL:
  [One of: "No action needed — merge quality confirmed for DSS" /
   "FORGE to fix [N] HIGH-FIX items before May 9 DSS readiness check" /
   "Presten response needed on [N] VERIFY items before ELO can clear demo teams"]
```

---

## Why This Matters

The 27,820 `team_merges` rows are the largest unchecked data quality risk in Evo Draw. Incorrect merges corrupt rating histories — every team that has played against a corrupted team carries some contamination in its own rating. This framework ensures the audit is fast (decision tree eliminates free-form judgment), consistent (same rules for every row), and auditable (every decision recorded with type and severity). Without it, the merge audit is an open-ended manual process. With it, ELO can classify a batch in one session and SENTINEL can review the results without reading ELO's reasoning from scratch.

---

## References

- `Rankings/Team Merges Audit — Presten Execution Package.md` — source of audit rows ELO classifies
- `Rankings/Pre-DSS Team Merge Audit — U13G.md` — targeted U13G demo team merge verification
- `Rankings/Merge Audit Triage Framework — May 2026.md` — earlier triage work; this framework supersedes it for systematic classification
- `Rankings/Rank Bands — Demo Examples Document.md` — demo team names for demo team status check
- `Rankings/Club Rankings Reference Spot-Check Set.md` — additional reference clubs for demo team check

*ELO — 2026-04-26*
