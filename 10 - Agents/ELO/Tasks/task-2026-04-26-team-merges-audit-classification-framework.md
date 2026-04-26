---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Team Merges — Audit Classification Framework.md"
---

# Task: Team Merges — Audit Classification Framework

## Context

The Team Merges Presten Execution Package is filed (`Rankings/Team Merges — Presten Execution Package.md`, 2026-04-26). Presten runs the audit queries April 29 – May 5. The audit will surface rows from the 27,820 team_merges entries for human review.

What does not exist: a classification framework for what ELO does with those results. The execution package tells Presten how to run the queries and export the data. But when ELO receives the raw output — potentially hundreds or thousands of rows to review — there is no defined taxonomy for sorting them, no severity levels, no decision rules.

Without a classification framework, ELO's merge review is free-form judgment for every row. This is slow, inconsistent, and produces output SENTINEL cannot audit. With a framework, ELO applies defined rules: this merge is type X, classification Y, action Z.

This task: write the classification framework before Presten runs the audit. When results land, ELO immediately applies the framework instead of reasoning from scratch.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Team Merges — Audit Classification Framework.md`

---

### Section 1: Merge Type Taxonomy

ELO defines 4 merge types based on what team_merges represents:

**Type A — Same Team, Same Club, Name Variant**
Definition: Two records that represent the same team, same club, same age group — only the name differs (e.g., "FC Dallas U15 Girls" merged with "FC Dallas Girls U15").
Action: CONFIRM MERGE. This is the correct use of team_merges.
Confidence required: High — ELO confirms same club ID, same age group, same gender, name is clearly a variant.

**Type B — Same Club, Plausible Merge, Needs Verification**
Definition: Two records from the same club with overlapping age group/gender but non-obvious name relationship. Could be a correct merge (club changed team name) or an incorrect merge (two distinct teams).
Action: VERIFY BEFORE CONFIRMING. ELO flags for Presten review with specific question.
Confidence required: Cannot confirm without additional context. ELO describes the ambiguity.

**Type C — Cross-Club Merge (Likely Error)**
Definition: Two records from different club IDs merged together. This would combine rating histories across clubs.
Action: FLAG AS ERROR. Cross-club merges almost certainly inflate or deflate ratings by contaminating rating histories.
Severity: HIGH. Every cross-club merge found should be reported to SENTINEL.

**Type D — Same Club, Different Age Group Merge (Likely Error)**
Definition: Same club but different age groups merged — e.g., a U14G record merged with a U15G record.
Action: FLAG AS ERROR. Different age groups have different cal values and should never share rating history.
Severity: HIGH.

---

### Section 2: Severity Levels

| Severity | Definition | SENTINEL Action Required? |
|----------|------------|--------------------------|
| CONFIRMED | Type A merge, verified correct | No |
| VERIFY | Type B merge, needs human confirmation | Presten confirmation requested |
| HIGH — FIX | Type C or D error, affects ratings | Yes — SENTINEL flags for correction |
| HIGH — INVESTIGATE | Cannot classify without more data | Yes — ELO describes what data is needed |

---

### Section 3: ELO Classification Decision Tree

When ELO receives a merge row, apply in order:

```
Step 1: Do the two records share the same club_id?
  NO → Type C (Cross-Club) → Severity HIGH-FIX → STOP
  YES → continue

Step 2: Do the two records share the same age group and gender?
  NO → Type D (Age Group Error) → Severity HIGH-FIX → STOP
  YES → continue

Step 3: Are the two team names clearly variants of each other?
  YES (obvious abbreviation, punctuation, word order) → Type A → CONFIRMED → STOP
  UNCERTAIN → continue

Step 4: Do the teams have overlapping game history (same opponents, same dates)?
  YES → Type A → CONFIRMED → STOP
  NO or UNKNOWN → Type B → VERIFY → flag for Presten
```

---

### Section 4: Output Format

ELO files a classification table for all reviewed merges:

| merge_id | team_1_name | team_2_name | club_id_1 | club_id_2 | age_group | gender | Type | Severity | ELO Determination | Action Required |
|----------|-------------|-------------|-----------|-----------|-----------|--------|------|----------|-------------------|-----------------|

**Batch filing:** ELO files this table in batches as results arrive from Presten's audit sessions. Do not wait until all rows are reviewed — file each batch within one session of receiving results.

---

### Section 5: Rating Impact Assessment Protocol

For every Type C or D error found, ELO performs a quick rating impact assessment:

1. **Identify affected teams:** Which teams are involved in the incorrect merge?
2. **Estimate contamination period:** How many games are in the shared rating history?
3. **Severity of impact:** More than 20 games contaminated = HIGH. Fewer = MEDIUM.
4. **Fix required:** Does the merge need to be split? Does a re-recompute follow? ELO states what fix is needed and who owns it (ELO or FORGE).

This assessment is filed as a one-paragraph note next to each HIGH-FIX row in the classification table.

---

### Section 6: Summary Report Template

After all merge batches are classified, ELO files a one-page summary:

```
Team Merges Audit — Classification Summary
Date: [date]
Rows reviewed: [N] of 27,820
Batches from Presten: [N]

Classification breakdown:
  Type A (CONFIRMED): [N] — [%]
  Type B (VERIFY): [N] — [%]
  Type C (Cross-Club Error): [N] — [%]
  Type D (Age Group Error): [N] — [%]

HIGH-FIX items: [N]
  - [list each cross-club or age-group error with affected team names]

VERIFY items pending Presten response: [N]
  - [list top 5 ambiguous merges by potential rating impact]

ELO confidence in remaining 27,820 − [N] unreviewed rows: [high / medium / low]
Reason: [one sentence]

Recommended next step for SENTINEL:
```

---

## Quality Standard

- The decision tree in Section 3 must be deterministic — every row produces exactly one Type classification without ELO judgment calls beyond the defined steps.
- Section 4 output table format must be used for all filed batches. Not free-form narrative.
- Section 5 rating impact assessment must be completed for every HIGH-FIX item before ELO files the batch. Do not defer impact assessment to a later session.
- Section 6 summary is filed when at least 80% of Presten's provided rows are classified, or by May 5, whichever comes first.

## Why This Matters

The 27,820 team_merges rows are a known risk in Evo Draw's data quality. Incorrect merges corrupt rating histories for the affected teams — potentially affecting rankings for every team that has played against a corrupted team. SENTINEL has flagged this as a Current Priority since the Config was written. The audit will surface the actual error rate. This classification framework ensures ELO's review is fast (decision tree, not free-form), consistent (same rules for every row), and auditable (every decision recorded with type and severity). Without it, the merge audit is a manual process that takes 3x longer and produces output SENTINEL cannot verify.

## References

- `Rankings/Team Merges — Presten Execution Package.md` — source of audit rows
- `Rankings/Team Merges Pre-DSS Audit.md` — audit scope and query design
- `Rankings/Dedup Strategy.md` — full team_merges context and the 27,820 count
- `Rankings/Merge Audit Triage Framework.md` — earlier triage work ELO can reference
