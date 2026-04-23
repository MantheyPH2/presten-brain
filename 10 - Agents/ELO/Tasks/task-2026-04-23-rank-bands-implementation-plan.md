---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
status: completed
completed: 2026-04-23
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Implementation Plan.md"
priority: high
---

# Task: Write Implementation Plan for Rank Bands (DB → API → Frontend)

## Context

Yesterday ELO delivered the Rank Bands Design (`02 - Tiger Tournaments/Projects/Rankings/Rank Bands Design.md`). The design is approved. Thresholds are set (Gold ≥1500, Silver 1350–1499, Bronze 1200–1349, Red 1050–1199, Blue 900–1049, Green <900). The SQL view `rankings_with_bands` is specified. This task converts the design into a sequenced implementation plan Presten can execute in ~5.5 hours.

SENTINEL priority: Rank Bands are the fastest-to-ship competitive response feature. USA Rank has them. We don't. This must be demo-ready before DSS (May 16, 2026). The implementation estimate from the design was ~5.5 hours — that is the target.

Read before starting:
- `02 - Tiger Tournaments/Projects/Rankings/Rank Bands Design.md` — your design from yesterday. Source of truth.
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — rankings table structure, confirm column names
- `02 - Tiger Tournaments/Projects/Infrastructure/Server and Frontend.md` — existing rankings API endpoint, React frontend structure
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — Ranking Bands table (USA Rank's bands, for reference — ours use rating values not rank positions)

## What You Need to Produce

### 1. Pre-Flight Checklist
Before Presten writes a single line:
- Confirm `rankings` table has columns: `team_id`, `rating`, `rank`, `age_group`, `gender`, `last_game_date`
- Run distribution query from design doc — confirm actual median is near ~1050 (if median differs significantly, flag threshold adjustment)
- Confirm the existing `GET /api/rankings` endpoint structure (what fields does it currently return?)
- Confirm frontend framework (React? Vue? plain HTML?) and where the rankings display component lives

### 2. Step-by-Step Implementation Sequence

Each step: what to do, exact SQL/code, how to verify, time estimate.

**Phase 1 — Database (Target: ~1 hour)**
1. Create the `rankings_with_bands` SQL view — paste the exact CREATE VIEW statement from the design doc
2. Verify: `SELECT team_id, rating, rank_band, band_color FROM rankings_with_bands LIMIT 20`
3. Spot check: confirm at least some Gold, Silver, Bronze, Red, Blue, Green rows appear
4. Confirm inactive teams (last_game_date > 7 months ago) show `band: NULL` or `band: 'Inactive'` per design spec

**Phase 2 — API (Target: ~1.5 hours)**
5. Modify `GET /api/rankings` response to include `band` and `band_color` fields — query `rankings_with_bands` instead of `rankings`
6. Add optional `band` filter parameter: `GET /api/rankings?band=Gold&age_group=U14G`
7. Test: `curl "http://localhost:3000/api/rankings?age_group=U13G&limit=5"` — confirm `band` and `band_color` in response JSON
8. Test filter: `curl "http://localhost:3000/api/rankings?band=Gold&age_group=U13G"` — confirm only Gold teams returned

**Phase 3 — Frontend (Target: ~3 hours)**
9. Add band badge component — a colored pill/chip that displays the band name in the band color
10. Add band badge to rankings table row (next to team name or rank number)
11. Add band filter UI — a dropdown or button group (All / Gold / Silver / Bronze / Red / Blue / Green) that calls the API with `?band=` parameter
12. Test: load rankings page, confirm badges render in correct colors, confirm filter works

### 3. Code Artifacts (Complete and Ready to Run)

Include copy-paste-ready code for:

**A. SQL CREATE VIEW** — `rankings_with_bands` — complete statement with the CASE block for band assignment and color, plus the inactivity check (NULL band for teams inactive 7+ months)

**B. Updated API handler** — modified rankings endpoint that queries `rankings_with_bands` and includes band/band_color in JSON output; also handles the `?band=` filter parameter

**C. Band badge component** — frontend component (match whatever framework is in use) that renders a colored pill with the band name. Props: `band` (string), `color` (hex). Handles NULL/inactive gracefully (renders nothing or "Inactive" in gray).

**D. Distribution check query** — the query Presten runs in pre-flight to confirm actual rating distribution matches expected thresholds. Should return count of teams per band per age group so Presten can sanity-check the percentages.

### 4. Threshold Contingency
If the pre-flight distribution query shows the median is significantly different from ~1050 (say, it's 950 or 1150):
- Provide a recalibration formula: how to shift all 6 thresholds proportionally
- Define "significantly different": more than ±75 points from ~1050 = recalibrate
- This should take no more than 10 minutes to adjust and re-run

### 5. Rollback Plan
- How to drop the view if it causes query performance issues
- How to remove band fields from API response without breaking frontend
- How to revert frontend to pre-band state

### 6. Definition of Done
Presten calls this shipped when:
- [ ] `rankings_with_bands` view exists and returns correct bands for known test teams
- [ ] API returns `band` and `band_color` in rankings response
- [ ] `?band=Gold` filter returns only Gold teams
- [ ] Frontend displays band badges in correct colors on the rankings page
- [ ] Filter UI allows filtering by band
- [ ] No existing functionality broken (original ranking table still works, other API endpoints unaffected)

## What Done Looks Like

Produce an implementation plan at:
`02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Implementation Plan.md`

The plan must be executable in a single ~5.5 hour session. Zero design decisions left open. All code artifacts complete.

## Success Criteria

- Plan written with all 6 sections
- All 4 code artifacts are complete (no placeholders, no TODOs)
- Presten can open the plan and start executing immediately
- Rank Bands are shippable before May 2, 2026 (per ELO's design doc estimate)
- Competitive gap vs USA Rank rank bands is closed upon completion
