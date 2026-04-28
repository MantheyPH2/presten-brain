---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: open
priority: high
due: 2026-05-10
deliverable: "02 - Tiger Tournaments/Projects/Data Sources/SnapSoccer May 17 — Build Session Spec.md"
---

# Task: SnapSoccer May 17 — Build Session Spec

## Context

The Non-GotSport Source Priority Matrix (filed 2026-04-27) confirmed SnapSoccer as the correct first non-GotSport build with a priority score of 18/24. The Go/No-Go recommendation (filed 2026-04-24) set the May 17 build date, contingent on the SincSports browser check returning ACCESSIBLE. The SnapSoccer Test Extraction Plan (filed 2026-04-27) covers the trial run. What does not exist: the actual build specification that FORGE follows on May 17 to produce a production-ready SnapSoccer ingestion pipeline.

On May 17, Presten and FORGE have a build session. Without this spec, that session begins with design work instead of execution — which has historically added 2–3 hours of overhead and risk of mid-session scope drift. This spec converts May 17 from a design+build session into a pure build session.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Data Sources/SnapSoccer May 17 — Build Session Spec.md`

---

### Section 1: Platform Profile

Confirm the following from the Go/No-Go recommendation and test extraction plan:

| Field | Value |
|-------|-------|
| Platform | SincSports (soccer.sincsports.com) |
| Access method | Public web — no auth required (ACCESSIBLE confirmed via browser check) |
| Data format | HTML tournament result pages, JSON endpoints (if discovered), or CSV exports |
| TID structure | Tournament IDs (TIDs) in URL path — `soccer.sincsports.com/[TID]/...` |
| Estimated active TIDs (SnapSoccer scope) | ~75 (per TID Coverage Estimate — confirm via pre-build SQL before May 17) |
| Estimated net new games | 5,000–7,000 (pending pre-build SQL to sharpen) |
| Crawl runtime estimate | < 2 minutes per full TID discovery pass |

If the browser check or test extraction has produced revised estimates by the time FORGE writes this spec, update these values.

---

### Section 2: Extraction Logic

Define the exact extraction approach FORGE will implement. This section must be specific enough that FORGE can open a code editor on May 17 and begin building without a design discussion.

#### 2a. TID Discovery

How does FORGE enumerate the complete set of active SnapSoccer TIDs?

- Source of the TID list: Is there a SnapSoccer tournament directory page? Or does FORGE derive TIDs from game records already in the DB? Or from the test extraction plan's methodology?
- Scraping approach: HTTP GET + HTML parsing, or a known JSON endpoint?
- Deduplication: How does FORGE avoid re-queuing TIDs that have already been ingested?

Describe the discovery step as a pseudocode sequence. Example:
```
1. Request soccer.sincsports.com/tournaments
2. Extract all tournament hrefs matching pattern /[A-Z0-9]{8,}/
3. Filter to soccer-category tournaments (exclude baseball, softball, etc.)
4. Return list of TIDs
```

#### 2b. Game Data Extraction

For each TID:
- URL pattern for results page: `soccer.sincsports.com/[TID]/results` (or equivalent — confirm from test extraction)
- Fields to extract: home team, away team, home score, away score, game date, division/age group, tournament name
- Parsing approach: CSS selector or JSON path for each field
- Handling of empty or cancelled games: skip, mark as played with null score, or flag?

#### 2c. Team Name Normalization

SnapSoccer team names will not match GotSport team names directly. Define the approach:
- Does FORGE attempt fuzzy match against existing team names in the DB?
- Or does FORGE insert raw SnapSoccer names and let ELO's team merge process handle normalization?
- If fuzzy matching: what confidence threshold before inserting a match vs. flagging for manual review?

If the decision is to let ELO handle normalization, state this explicitly so ELO knows to expect new unmatched team names in the merge queue after May 17.

---

### Section 3: Schema Mapping

Map SnapSoccer extracted fields to the games table schema:

| SnapSoccer Field | games Table Column | Transformation Required |
|-----------------|-------------------|------------------------|
| tournament_name | event_name | none |
| home_team | home_team_raw | none (raw insert; merge later) |
| away_team | away_team_raw | none (raw insert; merge later) |
| home_score | home_score | integer cast |
| away_score | away_score | integer cast |
| game_date | game_date | date parse (format: confirm from test extraction) |
| division | age_group, gender | parse: "U15 Girls" → age_group=U15, gender=F |
| TID | source_event_id | string |
| (derived) | source | 'snapsoccer' |
| (derived) | event_tier | 'snapsoccer' (or correct tier — confirm with ELO) |

**Note for FORGE:** Confirm the `event_tier` value for SnapSoccer games with ELO before May 17. The League Hierarchy must have a calibration value for 'snapsoccer' or the equivalent tier classification for SnapSoccer tournaments before any games can be used in rating computation.

---

### Section 4: Deduplication Strategy

SnapSoccer data may contain games already in the DB from GotSport (if teams played SnapSoccer tournaments that are also tracked in GotSport). Define the dedup approach:

- Primary key for deduplication: `(home_team_id, away_team_id, game_date, home_score, away_score)` — or a SnapSoccer game_id if one exists in the HTML
- On conflict: skip (do not re-insert); log the skipped count per TID
- FORGE files dedup rate in the post-build briefing: `X games extracted, Y already in DB, Z new games inserted`

---

### Section 5: Build Session Checklist

Linear checklist for May 17. Presten opens this document and follows from top to bottom.

```
Pre-Build Gate (5 minutes):
[ ] SincSports browser check confirmed ACCESSIBLE (from FORGE task-2026-04-27-snapsoccer-test-extraction-plan)
[ ] event_tier value for SnapSoccer confirmed with ELO and added to League Hierarchy
[ ] Pre-build SQL run: net new game estimate updated from TID Coverage Estimate range to point estimate
[ ] DB backup completed before any new source insertion

Step 1 — TID Discovery Module (45–60 minutes):
[ ] Write and test TID discovery function
[ ] Run against soccer.sincsports.com; confirm TID count in expected range (~75 ± 20)
[ ] Log TID list to file

Step 2 — Game Extraction Module (60–90 minutes):
[ ] Write extraction function for a single TID (test with 2–3 TIDs from test extraction plan)
[ ] Verify field mapping against schema (Section 3 above)
[ ] Run full extraction across all discovered TIDs
[ ] Log: total records extracted, parse errors, empty games skipped

Step 3 — Insertion + Dedup (30 minutes):
[ ] Run insertion with dedup logic (Section 4 above)
[ ] Log: new games inserted, duplicates skipped, insertion errors
[ ] Run Q1 verification: game count by source before vs. after

Step 4 — First-Run Validation (15 minutes):
[ ] Run 3 validation queries (see Section 6)
[ ] Compare actual new game count against pre-build estimate (Section 1)
[ ] GREEN: within ±30% of estimate → file FORGE Verification Note
[ ] YELLOW: 50–70% of estimate → identify TIDs with low counts; file investigation note
[ ] RED: < 50% of estimate or 0 new games → hold; flag to SENTINEL before proceeding
```

---

### Section 6: Validation Queries

Three queries Presten runs immediately after insertion to confirm the build succeeded:

**Q1 — New Game Count by Source**
```sql
-- Returns count of games with source = 'snapsoccer'
-- Expected: N games (per pre-build estimate ± 30%)
```

**Q2 — Age Group / Gender Distribution**
```sql
-- Returns age_group, gender, count(*) for snapsoccer games
-- Expected: distribution across U13–U17, both genders
-- Flag: if any expected age group shows 0 → possible division parse failure
```

**Q3 — Date Range Check**
```sql
-- Returns MIN(game_date), MAX(game_date) for snapsoccer games
-- Expected: dates fall within current season window (August 2025 – present)
-- Flag: if max date > today → date parse error (future dates)
```

FORGE writes the actual SQL for these queries as part of this task.

---

### Section 7: Post-Build Handoff to ELO

After a GREEN first-run validation:

- FORGE files a SnapSoccer Build Completion Note in the next briefing: `"SnapSoccer insertion complete. N new games inserted. Age group distribution: [summary]. event_tier: [value]. Team name normalization: [approach — raw insert / partial fuzzy match]. ELO: [N] new unmatched team names in merge queue."`
- ELO adds unmatched SnapSoccer team names to the team merge workflow
- SENTINEL authorizes the SnapSoccer source as active in the next coverage review

---

## Definition of Done

- Spec written at `Data Sources/SnapSoccer May 17 — Build Session Spec.md`
- All 7 sections present
- Extraction logic (Section 2) is pseudocode-specific — FORGE can implement without additional design decisions
- Schema mapping (Section 3) has all columns including event_tier (confirmed with ELO)
- Build Session Checklist (Section 5) is copy-paste ready for May 17
- Validation queries (Section 6) are written SQL, not placeholders
- Summary in briefing: "SnapSoccer May 17 Build Spec written. Extraction approach: [method]. Estimated new games: [N]. event_tier pending ELO confirmation: [YES/NO]. Spec ready for May 17 execution: YES."

## Why This Matters

The Non-GotSport Priority Matrix confirmed SnapSoccer is the right first build. The test extraction plan proves it's accessible. Now the question is execution quality on May 17. Without a build spec, May 17 is a design session masquerading as a build session — FORGE and Presten make schema decisions under time pressure, miss the dedup case, and file a partial insertion that needs rework. With this spec written and reviewed by SENTINEL before May 17, the session is a checklist execution. FORGE delivers a production-ready SnapSoccer ingestion in one session.

## References

- `Data Sources/Non-GotSport Source Priority Matrix — April 2026.md` — confirms SnapSoccer priority
- `Data Sources/SnapSoccer Go-No-Go Recommendation — April 2026.md` — go decision and access confirmation gate
- `task-2026-04-27-snapsoccer-test-extraction-plan.md` — test methodology and TID discovery
- `Data Sources/SnapSoccer-TID-Coverage-Estimate-2026-04-27.md` — net new game estimates
- `Rankings/League Hierarchy.md` — event_tier values (ELO must add 'snapsoccer' entry)
