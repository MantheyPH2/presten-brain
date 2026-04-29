---
type: directive-completion
parent_directive: directive-2026-04-28-dss-bracket-assignment.md
from: Claude Code (night-of executor)
date: 2026-04-28
time: "18:45 PT"
status: completed
---

# Directive Completion Note — DSS Bracket Assignment First Pass

Presten asked for this done tonight rather than waiting on the next agent cycle. Claude Code executed against the directive directly.

## Deliverables filed

All in `02 - Tiger Tournaments/Projects/Events/Dallas Spring Showcase/Brackets/`:

1. `01 — Game Coverage Report.md` + `01 — Game Coverage.csv`
2. `02 — Bracket Assignments.md` + `02 — Bracket Assignments.csv`
3. `03 — Flagged Teams for Presten Review.md` + `03 — Flagged Teams.csv`

## Headline numbers

- **513 teams** registered
- **Coverage:** 357 GREEN (70%) / 79 YELLOW (15%) / 37 RED (7%) / 40 MISSING (8%)
- **Flagged:** 283 total — 67 move-up, 83 move-down, 133 ranking-uncertain
- **Unassigned slotted:** 13 of 13 got a suggested division (10 with rating-based confidence, 3 manual due to missing rating). Bracket letter (A/B/C) still needs registrar input.

## Notable findings during execution

1. **Postgres was down on a stale postmaster.pid lock** (PID 561 was iCloudNotificationAgent, not postgres). Removed the stale lockfile and restarted. Standalone observation: Brew services postgres@17 had crashed without cleanup. May want a healthcheck.

2. **`games.home_team_id` / `games.away_team_id` store the GotSport `external_id` directly, NOT `teams.id`.** First version of this script joined the wrong column and reported 0 games for everyone. Documented in the Game Coverage Report. FORGE may want to either: (a) rename those columns to `home_external_id` for clarity, or (b) populate `teams.id` linkage to use internal ids consistently.

3. **`rankings.team_name + age_group + gender`** is the join key (no team_id column on rankings). This works but is fragile — name variations across registration platforms cause misses. ELO's calibration work would benefit from a `rankings.team_external_id` column or a join through `team_merges`.

4. **Empirical division-median ladder produced some directionally surprising flags** when divisions span game formats (11v11 vs 9v9 mixed in same Event Age). Documented as a known limitation in the Flagged Teams report. ELO's formal League Hierarchy Calibration would produce a more reliable v2.

5. **40 teams in CSV are not in `teams` table at all** — these are likely teams not yet ingested into Evo Draw. FORGE should investigate post-DSS whether they need a targeted scrape.

## Open follow-ups for the agent team

- **FORGE:** Investigate the 40 MISSING teams. Are they brand new GotSport teams that haven't been crawled?
- **ELO:** Re-run the flagging using the formal League Hierarchy Calibration once ECNL migration is settled. This v1 is the night-of pass; v2 should be canonical.
- **SENTINEL:** Review the 283 flagged teams + the 13 unassigned slotting suggestions before Presten makes movement decisions.
- **CHIEF:** Mark this directive resolved in the next briefing. Original directive items 1–3 (Game Coverage, Bracket Assignments, Flagged Teams) are all delivered.

## Match schedule generation

Out of scope for this pass per the directive. Next pass would: take the bracket assignments, apply the size-based format rules from `Bracketing Rules.md` (4-team RR+SF+F, 5-team RR+SF+F, 6-team two pools of 3 + cross-play, 8-team two pools of 4), and generate the match list. SENTINEL decides format for sizes 7/9/10 per the autonomy directive.

## Implementation notes for the agents

The script is at `~/Desktop/Projects/evo-draw/bracket-pass-1/run.js` if anyone wants to adapt or rerun. It's idempotent — re-running overwrites the deliverables. The CSV input lives at `Source/registrations-2026-04-29.csv` in the vault.

— Claude Code, 2026-04-28 18:45 PT
