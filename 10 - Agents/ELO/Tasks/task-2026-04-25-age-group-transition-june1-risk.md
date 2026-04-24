---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md"
---

# Task: June 1 Risk Assessment — Age Group Transition + ECNL Migration

## Context

Two significant changes land on or around June 1, 2026:

1. **ECNL platform migration:** Teams switch from TGS (The Gotsport System) to GotSport directly. FORGE already wrote a validation script for this (`task-2026-04-23-ecnl-migration-validation-script.md`), but that script captures data continuity (team IDs, game counts). It does not assess the rating system impact.

2. **Age group transition:** Teams that were U12 become U13, U13 become U14, etc. You wrote `task-2026-04-22-transition-migration-spec.md` and marked it completed. In your 17:06 briefing you flagged uncertainty about whether `Rankings/Age Group Transition Migration Spec 2026-06-01.md` was written in full, or just the task notes.

These two changes happening simultaneously is a compounding risk. The ECNL migration changes team identity (new platform IDs). The age group transition changes team classification. If the ranking system processes both at once with no safeguards, a team could be: (a) misidentified due to ID changes, and (b) incorrectly classified due to age group transition. The resulting ratings would be garbage.

June 1 is 39 days from today. DSS is May 16. The risk assessment must be in Presten's hands before implementation season is over.

## What You Need to Do

### Step 1: Audit the Existing Transition Migration Spec

Read `02 - Tiger Tournaments/Projects/Rankings/Age Group Transition Migration Spec 2026-06-01.md`.

If the document exists and is complete, verify it covers:
- [ ] Birth year to age group mapping logic for the June 1 transition
- [ ] Rating carryover formula (what percentage carries, what resets)
- [ ] RD reset values post-transition (from your Brier score work — RD should increase post-transition to reflect uncertainty)
- [ ] `birth_year_cohort` column usage — is it the source of truth for the new age group?
- [ ] How dual-system teams (teams that exist in both old and new age groups briefly) are handled
- [ ] Order of operations: does the age group transition run before or after the nightly Glicko compute?

If the document does not exist, or if any of these six items are not covered, note the specific gaps. **Do not re-write the entire spec** — identify what's missing and write only those sections.

### Step 2: ECNL Migration — Rating System Impact Analysis

FORGE's ECNL validation script checks data continuity. Your job is to assess the rating system implications. Specifically:

**Risk A: ECNL team ID discontinuity**
When ECNL teams move from TGS to GotSport, they may get new team IDs in our system. If our dedup logic doesn't catch this, the team's Glicko history is orphaned — the old record accumulates no new games, the new record starts at default rating ~1000. A previously Gold-band team would become unranked overnight.

Write the detection query:
```sql
-- Find ECNL teams with games in the last 90 days (active) that
-- appear in the teams table under both a TGS-style source_id and
-- a GotSport-style source_id, without a corresponding team_merges record
```
(Adapt to actual schema — describe what the query should accomplish if you don't know the exact column names.)

**Risk B: Simultaneous ID change + age group transition**
A team whose platform ID changes on June 1 AND whose age group changes on June 1 has two reasons to be mis-classified. Document the specific scenario:
- Team is U12G ECNL in TGS
- June 1: becomes U13G ECNL in GotSport
- Our system sees: new GotSport ID (new record) + U13G classification (correct) + no rating history (because the TGS record was U12G)
- Result: the team starts at ~1000 rating in U13G even if it was Gold U12G

Is there a merge path that preserves the rating through both the ID change and the age group transition simultaneously? Write the spec for this edge case.

**Risk C: Calibration values for "age group changed" teams**
Your transition migration spec includes K-factor and RD adjustments post-transition. Do those adjustments apply correctly if a team also changed its platform source? Or does the source-change pipeline run first and reset the team before the calibration adjustments are applied?

### Step 3: Write the June 1 Risk Matrix

Produce a risk matrix covering all identified risks:

| Risk | Likelihood | Impact | Detection Method | Mitigation |
|---|---|---|---|---|
| ECNL ID discontinuity → orphaned history | High | High | Pre-migration baseline query | Merge TGS→GotSport IDs before June 1 |
| Age group + ID change → double-reset | Medium | High | ... | ... |
| Calibration ordering bug | Low | Medium | ... | ... |
| (add all others you identify) | | | | |

For each risk rated High/High: write the minimum viable mitigation. This is not a full implementation spec — it's enough to unblock Presten in deciding whether to intervene.

### Step 4: Minimum Viable Patch List

Based on the risk matrix, list the minimum code changes needed before June 1 to prevent data corruption. For each:
- What breaks without this patch
- What the patch does (one sentence)
- Who owns it (ELO spec, FORGE implementation, or Presten direct)
- Estimated effort

Prioritize: if Presten can only do one thing before June 1, what is it?

## What Done Looks Like

Deliverable: `02 - Tiger Tournaments/Projects/Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md`

The document must:
- Confirm whether the existing transition migration spec is complete or flag specific gaps
- Assess rating system risks from the ECNL platform migration (not just data continuity — ELO's domain)
- Document the compounding risk scenario (ID change + age group transition simultaneously)
- Produce a risk matrix with mitigation for High/High risks
- Deliver a prioritized, minimum viable patch list

Format: match the level of rigor in `Calibration Production Runbook — May 2026.md`. Specific steps, named owners, concrete estimates.

## Success Criteria

- Transition migration spec gap audit is complete (document confirmed full or gaps identified and filled)
- ECNL ID discontinuity risk is analyzed with a detection query spec
- The compounding risk scenario (ID + age group change) is documented with a mitigation path
- Risk matrix is present with at least 4 risks rated
- Minimum viable patch list is delivered with prioritization
- Document is actionable before May 16 (Presten reviews it at DSS or immediately after)

## Why This Is ELO's Task

The ECNL migration data continuity is FORGE's domain (pipeline, ID matching, dedup). The rating system implications — what happens to a team's Glicko rating when its identity changes, and how the age group transition calibration interacts with that — is ELO's domain. These two systems intersecting on June 1 is the risk. FORGE sees the data. ELO sees the math. This document is the bridge.
