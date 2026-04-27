---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: open
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Calibration Values — League Hierarchy Reconciliation.md"
---

# Task: Calibration Values — League Hierarchy Reconciliation

## Context

The League Hierarchy document defines calibration values for every event tier. ELO's computation engine uses those calibration values to weight each game's contribution to a team's rating. These two sources must agree exactly.

Any discrepancy — a tier value that's 90 in the League Hierarchy doc but 100 in the engine, or a tier that exists in the engine but is absent from the doc — is a silent accuracy bug. It produces wrong ratings without error messages, without briefing flags, and without any observable signal until someone manually compares a team's expected vs. actual rating.

The GA ASPIRE overcalibration case (currently being fixed April 28) was exactly this kind of silent discrepancy: the engine treated GA ASPIRE events as `ga` tier (cal value 100) when the correct tier was `ga_aspire` (lower value). SENTINEL cannot confirm this was the *only* such discrepancy without a full reconciliation.

This task produces the reconciliation document. It should be completed before May 9 (DSS go/no-go) so that SENTINEL can represent calibration accuracy to Presten with confidence.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Calibration Values — League Hierarchy Reconciliation.md`

---

### Section 1: Reconciliation Table

For every event tier in the ELO computation engine, compare against the League Hierarchy document:

| Tier Code | Cal Value (Engine) | Cal Value (League Hierarchy) | Match? | Notes |
|-----------|-------------------|------------------------------|--------|-------|
| ecnl | | | ✅ / ❌ | |
| mlsnext | | | ✅ / ❌ | |
| ga | | | ✅ / ❌ | |
| ga_aspire | | | ✅ / ❌ | |
| npl | | | ✅ / ❌ | |
| dpl | | | ✅ / ❌ | |
| [all other tiers] | | | | |

Add a row for every tier that exists in either source. If a tier appears in the engine but not the doc — or in the doc but not the engine — that is a finding.

---

### Section 2: Discrepancy Log

List every row from Section 1 where Match = ❌:

| Tier Code | Engine Value | Hierarchy Value | Magnitude of Difference | Affected Gender / Age Groups | Estimated Rating Impact |
|-----------|-------------|-----------------|------------------------|------------------------------|------------------------|
| | | | | | |

For each discrepancy:
- State which source should be treated as authoritative and why
- Estimate how many teams are affected (rough: which leagues/age groups use this tier?)
- Estimate the direction and magnitude of rating distortion caused by the discrepancy

If Section 2 is empty (all tiers match): state this explicitly. An empty discrepancy log is a positive finding.

---

### Section 3: Missing Tier Analysis

State any tier that exists in the League Hierarchy doc but has NO corresponding games in the DB (i.e., a tier defined in the hierarchy that ELO has never seen a game for):

| Tier Code | In League Hierarchy? | Games in DB? | Interpretation |
|-----------|---------------------|--------------|----------------|
| | | | |

A tier with no DB games is not necessarily a bug, but it should be accounted for. Possible interpretations: (a) games haven't been ingested yet for this source, (b) the source uses a different tier code than expected, (c) the tier is defined prospectively and has no games yet this season.

---

### Section 4: Reconciliation Verdict

One of three verdicts:

**CLEAN** — All engine cal values match League Hierarchy. No discrepancies. ELO certifies calibration accuracy as of this date.

**MINOR DISCREPANCY** — 1–2 mismatches found, low impact (< 5 pts estimated rating effect). Identified and corrected. ELO certifies calibration is accurate after corrections.

**AUDIT REQUIRED** — 3+ mismatches, or any mismatch with estimated impact > 10 pts. Flag to SENTINEL immediately. Document each discrepancy and recommended correction for Presten review.

---

### Section 5: Correction Actions (if applicable)

If any discrepancies were found, list the specific code changes required to align the engine with the League Hierarchy:

| Tier | Current Value | Correct Value | Change Location in Code | Change Type |
|------|--------------|---------------|------------------------|-------------|
| | | | | |

Do not apply corrections without SENTINEL sign-off if verdict is AUDIT REQUIRED. If verdict is MINOR DISCREPANCY, ELO may apply corrections and note them in the next briefing.

---

## Quality Standard

- Every tier that exists in either the engine or the hierarchy doc must appear in Section 1. Do not limit the audit to "common" tiers.
- The Reconciliation Verdict must be one of the three options. Do not use language like "mostly aligned" or "appears correct."
- If ELO cannot locate the calibration values in the engine (e.g., they are runtime-configured rather than hardcoded), state this and describe the lookup path. The audit must be possible to complete regardless of where the values live.
- Document the version of the League Hierarchy used (date of last edit) so future audits can detect if the hierarchy was updated after the engine.

## Why This Matters

SENTINEL's primary quality mandate is ranking accuracy. The GA ASPIRE fix was discovered because Girls GA teams were observably overcalibrated against ECNL teams — a sufficiently large distortion that it showed up in narrative review. Smaller distortions — a tier valued at 95 instead of 90, affecting one age group — will not surface in narrative review. Only a systematic reconciliation surfaces them. SENTINEL cannot represent calibration accuracy at May 9 DSS go/no-go without this document. One undiscovered discrepancy in a high-volume tier could distort 500+ team rankings by 10–20 points — invisible until someone compares against USARank or manual ground truth.

## References

- `Rankings/League Hierarchy.md` — authoritative calibration values
- `Rankings/GA Coverage Audit Results — 2026-04-23.md` — prior example of a tier discrepancy
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — uses ga_aspire cal value (confirm it matches engine)
