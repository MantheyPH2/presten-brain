---
type: directive
from: Presten
to: CHIEF
cc: [SENTINEL, FORGE, ELO]
date: 2026-04-28
priority: high
deadline: 2026-05-09
---

# DIRECTIVE: DSS Bracket Assignment — First Pass

Bracket the Dallas Spring Showcase. Per the full-autonomy directive (2026-04-28), CHIEF and SENTINEL break this down and execute. I'll review the deliverables.

## Inputs (filed in the vault)

- **Registration CSV:** `02 - Tiger Tournaments/Projects/Events/Dallas Spring Showcase/Source/registrations-2026-04-29.csv` (513 teams, current as of today)
- **Bracketing rules:** `02 - Tiger Tournaments/Projects/Events/Dallas Spring Showcase/Bracketing Rules.md` — read this first, it has the format rules, the flagging rules, and the deliverables.

## What I want

Three documents in `Events/Dallas Spring Showcase/Brackets/`:

1. **Game Coverage Report** — every registered team, GREEN/YELLOW/RED game-coverage status from the past 365 days. FORGE writes the SQL, Presten or ops runs it, FORGE/ELO synthesizes.
2. **Bracket Assignments** — the final team-to-bracket assignment for all 513 teams. Honor the registrar's existing A/B/C labels. Slot the 13 unassigned teams by best fit, document the rationale per team.
3. **Flagged Teams** — teams whose Evo Draw ranking suggests they're in the wrong Division (move up or move down). Flag only — do not move. ELO produces the ranking-band thresholds per Division per Event Age. Insufficient game data → "ranking-uncertain" flag, no direction.

Match schedule generation is **out of scope** for this pass.

## Suggested breakdown (CHIEF/SENTINEL adjust)

- **FORGE Task 1:** SQL query bundle for game coverage (last 365 days per team, by Team ID). Include the exact psql command + a small results-validation routine.
- **FORGE Task 2:** Once the query results land, produce the Game Coverage Report.
- **ELO Task 1:** Ranking-band thresholds per Division per Event Age, anchored to the existing League Hierarchy Calibration work.
- **ELO Task 2:** Mismatch detection — given coverage report + ranking bands, produce the flagged-teams list with directions.
- **FORGE+ELO Task 3:** Joint bracket assignment file — registrar's assignments preserved, unassigned slotted, all 513 teams accounted for.

## Constraints

- Agents in this vault cannot run psql directly. Spec the queries; flag any DB execution as a Presten task in the same way the archive dry-run works.
- Don't move a team the registrar already assigned. Flag, don't move.
- For bracket sizes I didn't specify (7, 9, 10), SENTINEL picks a sensible default and documents it. Don't queue this back to me unless it's genuinely contentious.
- Deadline May 9 — gives a week of buffer before DSS opens May 16.

## Why this matters

DSS is the platform's flagship managed event. The bracket pass is the first end-to-end test of the rankings driving real tournament decisions. If the flagging surfaces real mismatches, that's a feature — it validates the calibration work the team has been doing.

— Presten
