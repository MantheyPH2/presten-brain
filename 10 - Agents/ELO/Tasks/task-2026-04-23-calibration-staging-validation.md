---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
status: pending
priority: high
---

# Task: Stage and Validate Calibration Changes in Dev Environment

## Context

Your 2026-04-22 briefing delivered the full Brier score analysis and calibration fix proposals for U12, U13, U14, and U19 (see `Rankings/Calibration Validation 2026-04-22.md`). Presten's approval is pending via queue item `pending-2026-04-22-calibration-approval.md`. While waiting for approval, your own briefing's "If no response yet" plan identified the next step: stage the changes in a dev/staging environment and run held-out test set validation to confirm the projected Brier improvements before committing to production.

Do not wait for Presten to approve before doing this validation work. When approval comes, the validation should already be complete so implementation can proceed immediately.

## What to Do

### Step 1: Stage the Calibration Changes

Create a staging copy of `calibration.json` (do not modify production):

- File: `config/calibration-staging-2026-04-23.json`
- Apply the proposed changes from Section 3 of `Calibration Validation 2026-04-22.md`:
  - U12: RD floor 50 → 60
  - U13: K-factor 32 → 24, RD floor 50 → 75
  - U14: K-factor 32 → 28, RD floor 50 → 65
  - U19: K-factor 32 → 24
  - All other age groups: unchanged

### Step 2: Run Held-Out Test Set Validation

Your briefing proposed running a held-out test set validation against the new parameters before committing to production. Do this now.

**Held-out test set construction:**
- Use the last 90 days of game data (January 2026 — March 2026) as the test set
- Train the Glicko-2 model on data up to December 31, 2025, using both the current and proposed parameters
- Evaluate Brier scores on the held-out January–March 2026 games for each age group

**Expected output:** A comparison table:

| Age Group | Current Brier (held-out) | Proposed Brier (held-out) | Improvement | Meets Threshold? |
|-----------|--------------------------|---------------------------|-------------|-----------------|
| U12       | ~0.241                   | ?                         | ?           | ?               |
| U13       | ~0.312                   | ?                         | ?           | ?               |
| U14       | ~0.273                   | ?                         | ?           | ?               |
| U19       | ~0.238                   | ?                         | ?           | ?               |

**Critical:** The held-out Brier scores may differ from your retrospective projections in Section 2 of the Calibration Validation document. If any age group does NOT meet the ≤ 0.23 threshold under the proposed parameters on the held-out set, report it immediately. Do not paper over a failed validation.

### Step 3: MLS NEXT Event Name Pattern Classifier

Your briefing's "If no response yet" plan also identified building the MLS NEXT event name pattern classifier as work that should proceed regardless of the calibration decision.

Build `scripts/classify-mlsnext-events.js`:
- Input: `event_tiers` table, querying all events where `event_tier = 'mlsnext'`
- Logic: classify each event as `mlsnext_homegrown`, `mlsnext_academy`, or `mlsnext_unclassified` using the pattern expansion rules from Section 4 of `Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md`:
  - Homegrown: event name contains "Homegrown" OR >60% of participating teams are known MLS academies
  - Academy: event name contains "Academy Division", "Regional", or division number pattern
  - Extended Homegrown patterns: "Premier", "Showcase", "Fest", "National" combined with existing mlsnext tier
  - Extended Academy patterns: geographic region + division number
  - Unclassified: everything else
- Output: A classification result set showing counts per category and a sample of 20 events per category for manual spot-check
- Write results to: `Reports/mlsnext-event-classification-2026-04-23.md`

**Target coverage improvement:** Your analysis found 82% identifiable. The pattern expansion should bring this to 94-96%. Report the actual coverage achieved.

### Step 4: Document the `backfill-age-gender.js` Changes for Dual-System Age Groups

Your briefing flagged Section 4 of the migration spec (dual-system age group handling in `backfill-age-gender.js`) as work to document before implementation. Write a technical note documenting the specific code changes needed:

- Which functions in `backfill-age-gender.js` need to be modified
- The exact logic additions required (the school-year offset calculation, event_tier lookup, `cross_system_age_group` flag)
- What the before/after behavior looks like for a cross-system game example
- Estimated implementation time

Write to: `Reports/backfill-age-gender-changes-spec-2026-04-23.md`

## What to Read

- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Rankings/Calibration Validation 2026-04-22.md` — your own document; Section 2 has the projections to validate
- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md` — Section 4 on data gap and reclassification options
- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Rankings/Age Group Transition Migration Spec 2026-06-01.md` — Section 4 on `backfill-age-gender.js` changes

## What to Produce

1. `config/calibration-staging-2026-04-23.json` — staging calibration file with proposed changes
2. `Reports/calibration-held-out-validation-2026-04-23.md` — held-out test set Brier score comparison table with pass/fail for each age group
3. `scripts/classify-mlsnext-events.js` — working event classifier
4. `Reports/mlsnext-event-classification-2026-04-23.md` — classification results with coverage percentage
5. `Reports/backfill-age-gender-changes-spec-2026-04-23.md` — technical note on `backfill-age-gender.js` modifications

## Definition of Done

- The held-out validation either confirms all four age groups reach ≤ 0.23 under proposed parameters, or identifies which ones don't and proposes parameter adjustments
- The MLS NEXT event classifier achieves >90% classification coverage on existing MLS NEXT events
- The backfill-age-gender changes are documented with enough specificity that implementation can proceed without re-analysis

## Timing Note

The target application window is May 18-20 (after DSS concludes). That is 27 days from now. The validation must be complete before Presten's approval can be meaningfully acted on. Do not be idle while waiting for approval — execute the validation now.
