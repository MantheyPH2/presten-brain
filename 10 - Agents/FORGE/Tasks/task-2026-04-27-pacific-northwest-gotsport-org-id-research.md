---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-05-02
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Pacific Northwest GotSport Org-ID Research.md"
---

# Task: Pacific Northwest GotSport Org-ID Research

## Context

The GotSport Coverage Map by State (filed 2026-04-26) and the VA/CO/AZ Org-ID Research (filed 2026-04-27) have identified and confirmed org-IDs for the Southeast (GA), Mid-Atlantic (VA), Mountain (CO), and Southwest (AZ) regions. The VA/CO/AZ document is staged as a Stage 2 expansion candidate.

The Pacific Northwest — Washington (WA), Oregon (OR), and Idaho (ID) — has not been specifically researched. Washington in particular is a significant youth soccer state with a strong ECNL Girls presence (Crossfire Premier, FC Alliance), active NPL programs, and several MLS Next clubs (Sounders Academy, PTFC, OL Reign Academy). Oregon has OBSA-affiliated programs and is a moderate-activity GotSport state. Idaho is smaller but has known GotSport-registered leagues through the Idaho Youth Soccer Association (IYSA).

If FORGE can identify WA, OR, and ID org-IDs on GotSport, these represent clean Stage 2 expansion targets with known high-activity leagues — ready to add to the Stage 2 config as soon as the May 1 pipeline is confirmed healthy.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Pacific Northwest GotSport Org-ID Research.md`

---

### Section 1 — WA State

Target organizations to research (all should be findable as GotSport org entities):

| Organization | Type | Notes |
|-------------|------|-------|
| Washington Youth Soccer (WYS) | State association | Primary GotSport licensee for WA |
| Pacific Northwest Soccer League (PNSL) | Regional league | Potential separate org-ID |
| Crossfire Premier SC | Club | Top ECNL club in WA |
| Seattle Sounders FC Academy | MLS Next | |
| PTFC Cascade (Portland-based, WA affiliate) | MLS Next-adjacent | |

For each organization: find the GotSport org-ID (numeric). Research method: GotSport public search, known tournament pages, or cross-reference with existing game data in Evo Draw where these teams already appear.

State whether each org-ID is likely a **source** (the org runs leagues/events FORGE should crawl) vs. a **team-only** entry (the org is a club with no league administration function — not useful as a crawl target).

---

### Section 2 — OR State

Target organizations:

| Organization | Type | Notes |
|-------------|------|-------|
| Oregon Youth Soccer Association (OYSA) | State association | |
| Oregon Club Soccer (OCS) | Regional league | |
| FC Portland | ECNL club | |
| Portland Timbers Academy | MLS Next | |
| OL Reign Academy (OR affiliate) | MLS Next-adjacent | |

Same format as Section 1. Confirm whether OYSA operates directly on GotSport or uses a different scheduling platform.

---

### Section 3 — ID State

Target organizations:

| Organization | Type | Notes |
|-------------|------|-------|
| Idaho Youth Soccer Association (IYSA) | State association | |
| Idaho Club Soccer (ICS) | Regional league | Smaller activity volume expected |

Note expected game volume for ID: FORGE should flag if the expected annual game count is < 500 — that may make a standalone IYSA org-ID a low-priority Stage 2 addition.

---

### Section 4 — Stage 2 Candidates Summary

After researching all three states, produce a ranked list:

| State | Org | Org-ID | Confidence | Priority | Reason |
|-------|-----|--------|-----------|---------|--------|
| WA | WYS | [TBD] | High/Med/Low | Stage 2 / Stage 3 / Skip | |
| OR | OYSA | [TBD] | | | |
| (etc.) | | | | | |

Prioritization criteria:
- **Stage 2:** Org-ID confirmed, expected game volume ≥ 500/year, no crawler conflicts known
- **Stage 3:** Org-ID found but not confirmed, or volume uncertain
- **Skip:** Org doesn't use GotSport, or is a club-only entry

---

### Section 5 — Config Staging

For each org-ID classified as Stage 2: pre-populate the config entry in the same format as the VA/CO/AZ Org-ID Research document. This way, Stage 2 authorization → immediate config deployment with no additional research.

---

## Definition of Done

- All 5 sections present
- WA and OR have ≥3 orgs researched each, with org-ID status (found/not found) recorded
- ID has ≥1 org researched
- Stage 2 Candidates Summary table is filled (even if most entries are "not found" or "Skip")
- Any confirmed Stage 2 org-IDs are pre-formatted for config staging in Section 5

## Why This Matters

SENTINEL's mandate includes finding and filling all coverage gaps. The Pacific Northwest is one of the highest-activity youth soccer regions in the country with significant ECNL, MLS Next, and NPL activity — and it is currently unresearched in the org-ID expansion program. This research fills that gap and creates ready-to-deploy Stage 2 candidates, reducing the time from "Stage 2 authorized" to "PNW games ingested" to a single config deploy.

## References

- `Infrastructure/GotSport Coverage Map by State.md` — current coverage inventory; PNW will appear as a gap
- `Infrastructure/VA CO AZ GotSport Org-ID Research.md` — format template for this document
- `Infrastructure/Stage 2 Expansion — Authorization Criteria.md` — criteria PNW orgs must meet for Stage 2
- `Infrastructure/GotSport Org-ID Master Reference.md` — existing org-ID registry; cross-check for duplicates
