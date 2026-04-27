---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-26
priority: high
due: 2026-05-02
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Research — VA, CO, AZ.md"
---

# Task: GotSport Org-ID Research — Virginia, Colorado, Arizona

## Context

FORGE's April 26 15:46 briefing (GotSport Coverage Map work) explicitly identified three high-priority states that are not in any FORGE research queue and not in the browser session plan:

- **Virginia** — DC suburb market, estimated high game volume, likely GotSport
- **Colorado** — Growing major market, Colorado Soccer Association
- **Arizona** — Large youth market, year-round play

The browser session plan covers: TX (Stage 1), CA NorCal, CA SoCal, FL, NY-metro, NJ, WA, PA-East, USL Academy, EDP, Elite 64, NAL. VA, CO, and AZ were identified as "natural Stage 3 additions after browser-session entities are processed."

SENTINEL is assigning this now — not after the browser session — because the research work (confirming GotSport presence, estimating game volume, identifying org-ID lookup path) is independent of the browser session and can be completed this week. When the browser session eventually expands to Stage 3, FORGE should have this ready immediately rather than starting from scratch.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Research — VA, CO, AZ.md`

---

### Section 1: Research Summary Table

For each of the three states:

| State | Primary Association | GotSport Presence (Confirmed/Likely/Unconfirmed) | Estimated Annual Game Volume | Org-ID Lookup Path | Research Source |
|-------|--------------------|-------------------------------------------------|-----------------------------|--------------------|----------------|
| Virginia | Virginia Youth Soccer (VYSA) | | | | |
| Colorado | Colorado Soccer Association (CSA) | | | | |
| Arizona | Arizona Soccer Association (AzSA) | | | | |

GotSport presence can be confirmed via:
- `system.gotsport.com` org search (requires browser session)
- League documentation referencing GotSport as platform
- Tournament listings that use GotSport registration links
- Cross-reference with existing FORGE source research from similar state associations

For this task, "Likely" is an acceptable finding if the browser lookup hasn't been done — state the evidence basis.

---

### Section 2: Per-State Research Detail

For each state, one block covering:

**Association and league structure:**
- Primary state association name and acronym
- Major leagues in this state (Club, ECNL-Regional, NPL, etc.)
- Age groups served (focus on U9–U19)

**GotSport presence evidence:**
- Tournaments or leagues confirmed on GotSport platform
- Any documentation, registration URLs, or public references
- If platform is something other than GotSport, name it

**Game volume estimate:**
- Number of registered teams (if findable from public records or soccer association websites)
- Estimated games per team per season
- Rough total estimate (order of magnitude is fine: "~3,000–8,000 games/year")

**Org-ID lookup status:**
- If org-ID is known: state it
- If org-ID requires browser session: confirm this and add to Section 2 of the Org-ID Master Reference
- If state association is not on GotSport: state alternative platform and move to Section 3 of Master Reference

---

### Section 3: Priority Recommendation

After completing Section 2, state FORGE's recommended priority order for adding these three states to the Stage 3 research queue:

1. First priority: [State] — Reason: [one sentence]
2. Second priority: [State] — Reason: [one sentence]
3. Third priority: [State] — Reason: [one sentence]

This recommendation informs SENTINEL's task ordering when Stage 3 browser sessions are scheduled.

---

### Section 4: Browser Session Addition Recommendation

For any state confirmed as GotSport-present but without an org-ID, state whether FORGE recommends adding it to an **existing** upcoming browser session or scheduling a dedicated Stage 3 session. Include estimated browser session time (minutes) for all three states combined.

---

## Quality Standard

- Do not leave GotSport presence as "Unknown" — either confirm it or state the evidence basis for "Likely." If no evidence is findable via public research, state that explicitly and recommend a browser session as the only path to confirmation.
- Game volume estimates are acceptable as ranges ("~5,000–12,000 games/year"). Do not leave them blank.
- If a state association is confirmed as NOT on GotSport, this is a useful finding — document the alternative platform and the implication for FORGE's ingestion roadmap.

## Why This Matters

The GotSport Coverage Map revealed 43 states with zero coverage. VA, CO, and AZ were identified as the three highest-priority uncovered GotSport states by FORGE's own analysis. The browser session plan will run out of capacity on its current 11-entity scope — having Stage 3 research pre-done means when Stage 2 entities are confirmed, Stage 3 can begin immediately. Without this research, SENTINEL cannot prioritize Stage 3 expansion, and coverage gaps in three major youth soccer markets persist indefinitely.

## References

- `Infrastructure/GotSport Coverage Map — States and Regions.md` — source of the VA/CO/AZ gap identification
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — update Section 2 or Section 3 with findings
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — current Stage 2 browser session scope (11 entities)
