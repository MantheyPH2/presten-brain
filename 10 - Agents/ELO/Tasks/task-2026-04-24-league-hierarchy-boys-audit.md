---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: medium
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Leagues/League Hierarchy.md (updated in-place)"
---

# Task: League Hierarchy — Boys Circuits Audit

## Context

ELO's 08:52 briefing noted: the Boys Calibration Gap Analysis references pulling Boys GA calibration values from `League Hierarchy.md`, but the file "may need a standalone write (currently only referenced in other docs, not confirmed as a standalone file)."

SENTINEL's Config references `[[League Hierarchy]]` as a key vault resource: "all leagues and calibration values." If this file doesn't exist as a standalone, the reference is broken and the calibration audit trail is incomplete.

Two issues to resolve:
1. **Does `League Hierarchy.md` exist?** If not, it needs to be created.
2. **If it exists, does it contain Boys circuit entries with calibration values?** The Boys Calibration Gap Analysis depends on these values being real and correct.

This is a research + vault-edit task. Complete it before April 28 (the Boys calibration session date).

## What to Do

### Step 1: Verify Whether League Hierarchy Exists

Check the vault for a file named `League Hierarchy.md` or similar (may be at `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy.md` or in a references folder). Also check for it under alternate names: "League Tiers", "Circuit Hierarchy", "Division Calibration".

**If found:** Read it and proceed to Step 2.

**If not found:** Create a minimal version as described in Step 3 below, then proceed.

### Step 2: Audit Boys Circuit Entries (If File Exists)

For every Boys circuit in the system, confirm the file contains:
- Circuit name (ga, ga_aspire, mlsnext_academy, mlsnext_homegrown, ecnl_boys, npl_boys, etc.)
- Current calibration value
- Whether the value is empirically validated or assumed

Specifically check:
- Is `ga_boys` (or the Boys equivalent of `ga`) listed separately from `ga_girls`? Or do Boys and Girls share the same GA calibration value?
- Is `ga_aspire_boys` listed? The GA ASPIRE fix for Girls changed the Girls value — if Boys uses the same value, the Boys GA ASPIRE calibration is also affected.
- Are any Boys circuits missing entirely from the hierarchy?

Create a table in your briefing:

| Circuit | Boys Cal Value | Girls Cal Value | Same? | Boys Value Source |
|---------|---------------|-----------------|-------|-------------------|
| ga | [X] | [Y] | yes/no | validated / assumed |
| ga_aspire | ... | ... | ... | ... |
| mlsnext_academy | ... | ... | ... | ... |
| ecnl_boys | ... | ... | ... | ... |

### Step 3: Create or Update the League Hierarchy File

**If the file does not exist:** Create `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy.md` with the following minimum content:

```markdown
---
title: League Hierarchy
tags: [calibration, rankings, evo-draw]
status: reference
---

# League Hierarchy — Calibration Values

> Calibration values by circuit, both genders. Updated as values are validated.

## Girls Circuits
| Circuit | Calibration Value | Status |
|---------|-------------------|--------|
| ecnl_girls | [value from Ranking Engine] | validated |
| ga | [value] | validated |
| ga_aspire | [value — note if GA ASPIRE fix has been applied] | validated (post-fix) / pending fix |
| mlsnext_academy | 135 | validated — see MLS NEXT Tier Split Calibration Spec 2026-04-24 |
| mlsnext_homegrown | 160 | validated |
| mlsnext_unclassified | 147 | interpolated |
| npl | [value] | [status] |

## Boys Circuits
| Circuit | Calibration Value | Status | Notes |
|---------|-------------------|--------|-------|
| ecnl_boys | [value] | validated / assumed | |
| ga (boys) | [value] | assumed (shared with Girls) | Needs Boys-specific validation |
| ga_aspire (boys) | [value] | assumed | If GA ASPIRE fix applied to Girls, Boys value TBD |
| mlsnext_academy | 135 | validated | Assumed same as Girls — confirm |
| ... | | | |
```

Pull all values from `Ranking Engine.md` and `Division Calibration.md`. Do not use placeholder values for circuits you have real data for.

**If the file exists and is incomplete:** Add the missing Boys circuit rows. Do not remove or change existing content.

### Step 4: Flag the GA ASPIRE Boys Implication

The GA ASPIRE Girls calibration fix is pending Presten execution (April 28). Determine: does the same miscalibration apply to Boys?

If Boys and Girls share the same GA ASPIRE calibration value: the Boys value is equally over-calibrated (~40pts) and should be fixed in the same session as the Girls fix. Add a note to the April 28 Escalation Reference Card (or flag to SENTINEL to update it).

If Boys and Girls use separate GA ASPIRE calibration values: confirm that the Boys value is correct and document why.

Write your finding as a one-paragraph flag at the bottom of the League Hierarchy document and in your briefing.

## Definition of Done

- Confirmed whether `League Hierarchy.md` exists (or created it)
- All Boys circuit entries confirmed or documented as missing/unknown
- Boys vs Girls calibration value comparison table in briefing
- GA ASPIRE Boys implication documented (same value = same fix needed / different value = documented reason)
- Summary in briefing: "League Hierarchy audit complete. Boys circuits: [summary]. GA ASPIRE Boys flag: [same fix needed / separate value, confirmed]."

## Why This Matters

ELO's Boys Calibration Gap Analysis references calibration values from `League Hierarchy.md` that may not exist. If Presten runs the Boys Option A queries and finds anomalies, SENTINEL will ask for the Boys calibration values. If they can't be found in the vault, the audit trail is broken and calibration decisions become guesswork. This task closes that documentation gap before it matters.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — source for calibration values
- `02 - Tiger Tournaments/Projects/Rankings/Division Calibration.md` — source for tier definitions
- `02 - Tiger Tournaments/Projects/Rankings/Boys Calibration Gap Analysis — April 2026.md` — the analysis that depends on this file
- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split — Calibration Spec 2026-04-24.md` — MLS NEXT values to include
