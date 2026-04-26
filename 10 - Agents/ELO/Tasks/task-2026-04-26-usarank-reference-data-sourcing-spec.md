---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Rankings/USA Rank Reference Data — Sourcing Spec.md"
---

# Task: USA Rank Reference Data — Sourcing Spec

## Context

ELO filed `Rankings/USA Rank Post-April-28 Comparison Execution Package.md` — 3 copy-paste queries comparing Evo Draw ratings against USA Rank for the same teams, plus a comparison table and verdict framework.

The execution package's comparison table has a standing open item: "SENTINEL provides USA Rank reference data." SENTINEL does not currently have this data indexed. It is not in the vault. The comparison cannot run without it.

This is a structural gap that has been open since April 25. The May 3 comparison window is 7 days away. If Presten has to collect USA Rank data cold on May 3, the comparison session will spend 30–60 minutes on data collection before the analysis can start. If the USA Rank data needs manual entry into a structured format, that adds more time.

This task: define exactly what USA Rank data is needed, in what format, and how Presten can collect it before May 3. ELO writes this spec; SENTINEL uses it to brief Presten on what to collect.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/USA Rank Reference Data — Sourcing Spec.md`

---

### Section 1: Data Requirements

Define exactly what data ELO needs from USA Rank to complete the comparison:

**Which age groups:** Which age groups does the USA Rank comparison cover? All age groups in the Evo Draw execution package, or a subset? List them explicitly.

**Which teams:** The execution package compares teams that appear in BOTH Evo Draw rankings and USA Rank. ELO should specify: does the comparison use a fixed list of "known crossover teams" (e.g., teams ELO has already identified as appearing in USA Rank's top 100)? Or does Presten need to look up teams dynamically from Evo Draw's top-25 lists?

**What data fields are needed per team:**
- USA Rank's ranking position (rank #)
- USA Rank's rating score (if they show one, or only rank?)
- Season/date the USA Rank data was captured (for staleness comparison)

**How many teams:** Give Presten a specific count — e.g., "top 20 teams per age group" or "25 teams that appear in both platforms." A bounded scope prevents a 3-hour data collection session.

---

### Section 2: Sourcing Options

Presten needs to know WHERE to get the data. ELO should specify options in order of preference:

**Option A: Vault Note**
Is there already a USA Rank data capture anywhere in the vault? Check `Competitors` note, `Rankings/` folder, any prior SENTINEL or FORGE briefing that mentioned USA Rank numbers. If it exists, reference the file path and confirm the data is current enough for a May 3 comparison.

**Option B: USA Rank Website**
Presten navigates to USA Rank directly and captures rankings. Specify:
- Which page(s) to visit (age group-specific ranking pages, not the homepage)
- Whether screenshot, copy-paste, or manual entry into a template is most efficient
- How long this takes realistically: ELO estimates, based on the number of age groups × teams per group

**Option C: ELO Extracts from Evo Draw First**
Presten runs Evo Draw's top-N teams for each age group first, then looks up those specific teams in USA Rank. This "anchor on Evo Draw" approach reduces USA Rank data collection to a targeted lookup rather than a broad export.

Specify which option ELO recommends and why.

---

### Section 3: Input Template

Pre-build the data entry template Presten fills in after collecting USA Rank data:

```
USA Rank Reference Data — Captured [date]
Source: USA Rank website, captured by: Presten

| Age Group | Team Name | USA Rank Position | USA Rank Score (if shown) | Notes |
|-----------|-----------|-------------------|--------------------------|-------|
| U13G | | | | |
| U14G | | | | |
| U15G | | | | |
| U16G | | | | |
| U17G | | | | |
```

This template is what Presten fills in and sends to ELO. ELO feeds it into the comparison execution package's comparison table.

---

### Section 4: Comparison Validity Notes

ELO should document what conditions make the comparison valid vs. invalid:

- **Rating vs. rank:** If USA Rank only shows rankings (not scores), ELO's comparison is Evo Draw rating vs. USA Rank rank — a rank-to-rating cross-check, not a score-to-score comparison. Is this still meaningful? ELO states explicitly whether a rank-only comparison is sufficient for the DSS narrative claim.
- **Staleness:** If USA Rank data is captured April 28–May 3 and Evo Draw data is post-April-28 recompute, the comparison is contemporaneous. Note if any older USA Rank data in the vault is still acceptable.
- **Coverage gap caveat:** USA Rank may not list teams that are only in ECNL (since USA Rank leans GotSport-adjacent). If fewer than 15 teams per age group appear in both platforms, the comparison may not be meaningful. ELO states the minimum overlap needed for a valid comparison.

---

### Section 5: Briefing Summary for SENTINEL

One paragraph ELO writes for SENTINEL to use when briefing Presten:

"To run the USA Rank comparison before May 3, Presten needs to collect [N] teams × [M] age groups from USA Rank, estimated [X] minutes at [URL pattern]. Recommended method: [Option A/B/C]. Data entry template is at [file path]. Send completed template to ELO. ELO returns comparison results within [N] hours."

---

## Quality Standard

- Section 1 must have specific numbers, not ranges: exactly which age groups, exactly how many teams.
- Section 2 must recommend exactly one option (not "it depends").
- Section 3 template must be complete — Presten can start filling it in immediately after this file is written.
- Section 5 must be copy-paste ready for SENTINEL's next briefing.

## Why This Matters

The USA Rank comparison is the only external validation of Evo Draw's ranking accuracy. For the DSS audience, "we rank more teams than USA Rank and our rankings correlate" is a stronger claim than any internal calibration metric. If this comparison can't run before May 9, the DSS coverage narrative loses its external benchmark. The data collection bottleneck has been open since April 25. This spec converts an open question ("what USA Rank data do we need?") into a 30-minute Presten data collection task with a defined end state.

## References

- `Rankings/USA Rank Post-April-28 Comparison Execution Package.md` — the execution package this spec unblocks
- `Rankings/USA Rank Delta Interpretation Spec.md` — ELO's framework for interpreting comparison results
- `Competitors` (vault note) — check for any existing USA Rank data before specifying new collection
- `Product & Planning/DSS Demo Spec — May 2026.md` — the coverage claim this comparison supports
