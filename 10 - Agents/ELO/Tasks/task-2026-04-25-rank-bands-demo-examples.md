---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Demo Examples Document.md"
---

# Task: Rank Bands — Demo Examples Document

## Context

Rank Bands has the highest DSS demo value rating (5/5). The feature infrastructure is complete: the implementation plan, post-April-28 validation spec, and view SQL exist. What does not exist: the specific named examples Presten needs to demonstrate the feature compellingly to a non-technical DSS audience.

Without named examples, the Rank Bands demo requires Presten to improvise live queries — picking a team name from memory, hoping the result is interesting, and explaining what the band means on the fly. The risk: a demo team that lands in the wrong band (a well-known club rated "Developing" due to data coverage, not actual quality) or a boring example that doesn't tell a story.

This document pre-selects the examples, pre-writes the lookup queries, and gives Presten a narrative to follow. SENTINEL reviews it before May 9.

## What to Produce

### File: `Rankings/Rank Bands — Demo Examples Document.md`

---

### Section 1: Band Overview Query (Demo Opening Shot)

One SQL query Presten runs at the start of the Rank Bands demo to show the full distribution. Should return: how many teams are in each band, by age group and gender.

```sql
-- ELO writes the actual query
-- Should return: age_group, gender, band_label, team_count
-- Order: by age_group, then by band rank (Elite → Low)
```

**Demo purpose:** Shows the audience the scale — "there are X teams in the Elite band across all age groups." Establishes that Rank Bands is a real distribution, not a list.

---

### Section 2: Named Examples by Band

For each of the 5 bands (Elite, High, Mid, Developing, Low), identify 2 representative teams with a strong demo story. Criteria for a strong demo example:
- Recognizable club name to a soccer-knowledgeable audience
- Band assignment that is *obviously correct* (no edge cases where a reasonable person would dispute it)
- Available for a U15G or U16G Girls lookup (highest demo-value age group per DSS Demo Feature Sequence Plan)

For each team:

| Band | Team Name | Age Group | Expected Rating | Why This Is a Good Example |
|------|-----------|-----------|----------------|---------------------------|
| Elite | [name] | [U__G/B] | [range] | [1 sentence] |
| Elite | [backup] | | | |
| High | [name] | | | |
| High | [backup] | | | |
| Mid | [name] | | | |
| Mid | [backup] | | | |
| Developing | [name] | | | |
| Developing | [backup] | | | |
| Low | [name] | | | |
| Low | [backup] | | | |

ELO selects these from current rankings data — the selection must be defensible, not random.

---

### Section 3: Individual Team Lookup Query

One parameterized SQL query Presten uses to pull up any named team during the demo:

```sql
-- ELO writes the actual query
-- Input: team name (fuzzy match preferred)
-- Returns: team_name, age_group, gender, current_rating, band_label, band_rank_in_age_group
-- Include: percentile within band, game count (data confidence signal)
```

**Demo purpose:** Presten types a club name and the audience sees the band, rating, and rank instantly.

---

### Section 4: Narrative Script (3 scenes)

**Scene 1 — Introducing Rank Bands** (30 seconds)
One paragraph. What do Rank Bands mean in plain language? What problem do they solve that a raw rating number doesn't?

**Scene 2 — The Overview Shot** (60 seconds)
Narration for running the Section 1 query. What does the distribution tell us? What should the audience take away from seeing N Elite teams and N Developing teams in a single table?

**Scene 3 — The Named Example** (90 seconds)
Walk through one Elite example team and one Mid example team side by side. What story does the contrast tell? Why does band assignment matter to a coach or club director?

---

### Section 5: Contingency — If April 28 Shifts Band Assignments

The April 28 GA ASPIRE fix will change ratings for Girls GA teams. Some teams may shift bands. For the 2 Primary examples (one Elite, one Mid): ELO notes whether the selected team has significant GA ASPIRE exposure. If they do, ELO provides a backup example from a non-GA-ASPIRE-dominant team as insurance that the band assignment is stable post-April-28.

---

## Definition of Done

- 10 named teams across 5 bands, all U15G or U16G Girls
- Individual lookup query is copy-paste ready and returns band, rating, rank, and game count
- Section 1 distribution query returns a readable multi-row result
- Section 4 narrative is complete prose, not bullet points
- Section 5 identifies which primary examples are post-April-28 stable

## Why This Matters

Rank Bands is the feature most likely to produce a "wow moment" for the DSS audience — someone recognizes a club and immediately understands that the band is accurate. Pre-selecting the examples is what makes that moment reliable instead of lucky. ELO knows the data; SENTINEL doesn't. This document transfers that knowledge into the demo pipeline before May 9.

## References

- `Rankings/Rank Bands — Post-April-28 Validation Spec.md` — validation methodology
- `Rankings/Rank Bands — Threshold Validation.md` — band boundary thresholds
- `Product & Planning/DSS Demo Feature Sequence Plan — May 2026.md` — demo sequencing context
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — which teams may shift bands post-April-28
