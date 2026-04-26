---
title: ECNL Migration — CP1 Fail Escalation Protocol
type: escalation-protocol
author: ELO
date: 2026-04-26
status: ready — active on April 29 if CP1 FAILS
related: "[[ECNL Migration — Option Comparison Matrix]]", "[[ECNL Migration — Option Selection Decision Gate]]", "[[April 29 ELO Session Master Script]]"
tags: [ecnl, migration, escalation, calibration, decision, evo-draw]
---

# ECNL Migration — CP1 Fail Escalation Protocol

> **Written:** 2026-04-26 — before CP1 runs.
> **Activates:** April 29, if Step 1 of the April 29 Session Master Script returns ECNL RL staleness > 30 days.
> **Option selection deadline:** May 10. Presten sign-off required by May 5 if this protocol activates.

---

## Section 1: CP1 Fail Definition

CP1 PASSES or FAILS based on one query. Run this at the start of the April 29 session (Step 1 of Master Script):

```sql
SELECT
  MAX(g.game_date) AS latest_ecnl_rl_game,
  CURRENT_DATE AS today,
  CURRENT_DATE - MAX(g.game_date) AS days_stale
FROM games g
JOIN events e ON e.id = g.event_id
JOIN teams t ON (t.id = g.home_team_id OR t.id = g.away_team_id)
WHERE e.event_tier = 'ecnl_rl';
```

**Interpret the result:**

| `days_stale` | Result | Option 1 Status |
|-------------|--------|-----------------|
| ≤ 30 | **CP1 PASS** — ECNL RL data is current. Option 1 remains viable. Continue evidence collection. | Active |
| 31–60 | **CP1 FAIL** — ECNL RL data is stale. Option 1 disqualified. This protocol activates. | Disqualified |
| > 60 | **CP1 FAIL** — Same action. ECNL RL source may be dead or disconnected. | Disqualified |
| NULL (no rows) | **CP1 FAIL** — No ECNL RL games in DB at all. Option 1 disqualified immediately. | Disqualified |

**There is no gray zone.** > 30 days = FAIL = Option 1 disqualified. Do not negotiate with the threshold.

---

## Section 2: Immediate Actions (Within the April 29 Session)

Take these four steps **before continuing with any other April 29 work** if CP1 FAILS:

### Action 1 — Update the Option Comparison Matrix

Open `Rankings/ECNL Migration — Option Comparison Matrix.md`. In the Running Verdict Tracker (Section 4):

Fill the CP1 row:
- **Run Date:** April 29
- **Actual Result:** "ECNL RL data stale by [N] days. Option 1 disqualified."
- **Option 1 Status:** `Disqualified — CP1 (ECNL RL staleness > 30 days)`
- **Option 2 Status:** `Active`
- **Option 3 Status:** `Active`

Also update the G2 gate row after the full Decision Tree runs (Step 2–4 of the April 29 script) — this is a separate action, not part of the CP1 immediate response.

### Action 2 — File a Same-Session Queue Item

Write to `10 - Agents/ELO/Queue/pending-2026-04-29-ecnl-cp1-fail.md`:

```yaml
---
type: agent-queue
agent: ELO
category: alert
priority: urgent
date: 2026-04-29
status: pending
answer: ""
---
```

Content:
> CP1 FAIL confirmed. ECNL RL data stale by [N] days (threshold: 30 days). Option 1 (No Re-tag) is disqualified — Option 1's convergence mechanism requires live ECNL RL game ingestion to work. Migration decision is now between Option 2 (Re-tag + rating cliff) and Option 3 (ecnl_legacy). Option 2 requires Presten sign-off on the rating cliff before the May 10 option selection deadline. Option 3 requires DDL authorization from SENTINEL and a defensible ecnl_legacy cal value. Request: Presten session to review Option 2 cliff implications before May 5. If no session confirmed by May 5, ELO defaults to Option 3 (ecnl_legacy) as the lower-risk non-cliff option.

### Action 3 — Flag in the April 29 Briefing

In the `flags:` section of `10 - Agents/ELO/Briefings/2026-04-29.md`:

```yaml
flags:
  - "ECNL CP1 FAIL — Option 1 disqualified (ECNL RL staleness > 30 days). Migration on Option 2/3 track. Queue item filed: pending-2026-04-29-ecnl-cp1-fail. Presten sign-off required by May 5."
```

### Action 4 — Continue April 29 Remaining Steps

CP1 FAIL does not halt the entire session. Continue with Steps 2–6 of the Master Script:
- Step 2: Post-fix calibration baseline (G1–G4 Decision Tree)
- Step 3: Girls Calibration Narrative
- Step 4: Prediction vs. Actual Assessment
- Step 5: CP2 (ecnl_verified column check)
- Step 6: Pre-May-1 Baseline Snapshot

Steps 7–8 (Boys Option A, conditional) proceed normally — they are independent of ECNL migration option selection.

---

## Section 3: Option 2 Implications

> Pre-written before April 29. This section exists so that when CP1 FAIL is confirmed, ELO does not need to analyze Option 2 under time pressure.

**What Option 2 means:** On June 1, run an UPDATE on `games` that re-tags all historical `ecnl` games for Path A teams from `event_tier = 'ecnl'` to `event_tier = 'ecnl_rl'`. Run a full ranking recompute afterward. Path A teams will earn a one-time downward rating shift as their historical wins are re-valued at the lower `ecnl_rl` calibration value (cal=55 vs. cal=120/130).

**Estimated rating cliff magnitude:** Based on the calibration analysis in `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` (Section 2):

| Path A Team Type | Historical ECNL Games | Estimated Cliff |
|------------------|-----------------------|-----------------|
| Average Path A team | ~20 games, 55% win rate | 40–80 pts downward |
| High-volume Path A team | 30+ games, high win rate | 80–100+ pts downward |
| Low-volume Path A team | < 10 games | 15–30 pts downward |

The calibration gap is 65 pts (Boys) and 75 pts (Girls) between the historical `ecnl` tier and the forward `ecnl_rl` tier. A team that earned most of its rating points from ECNL games will see the largest cliff; a team that has accumulated significant non-ECNL games will see a smaller cliff (those games are unaffected by the re-tag).

**Which teams are most exposed:** Girls Path A teams are slightly more exposed than Boys Path A teams (75-pt cal gap vs. 65-pt gap). High-volume ECNL competitors — teams with 25+ historical ECNL games — are most exposed. These are typically the top-ranked ECNL programs, which means the cliff is most visible at the top of the Girls rankings.

**What Presten needs to decide:** Option 2 is executable and technically sound. Presten's sign-off must acknowledge one of the following:
- (a) The 40–80 pt rating cliff is expected and acceptable for Path A teams transitioning from ECNL to ECNL RL. ELO will notify affected clubs if a communication mechanism exists.
- (b) Option 3 (ecnl_legacy) is preferred to avoid the cliff entirely, accepting the additional DDL complexity and ELO's estimated cal value for the legacy tier.

**ELO's current lean:** Option 2 is preferable to Option 3 if Presten is willing to accept the cliff. Option 3 requires a new `event_tier = 'ecnl_legacy'` DDL change, a cross-league-data-backed cal value ELO currently does not have (would require ELO to declare an estimate), and ongoing two-track maintenance. The Option 2 cliff is a one-time, predictable event that ELO can explain clearly to SENTINEL and Presten. Option 3's ongoing complexity is a higher long-term cost. **ELO recommends Option 2 if Presten approves the cliff. ELO switches to Option 3 if the cliff magnitude from actual CP1 data exceeds 100 pts for > 20% of affected teams.**

---

## Section 4: Option 3 Comparison

**Option 3 in one line:** Introduce `event_tier = 'ecnl_legacy'` for all pre-migration ECNL games, calibrated at a blended value between `ecnl` and `ecnl_rl` cal values.

**What this avoids:** The rating cliff. Historical ECNL games are not re-valued to `ecnl_rl` — they receive a new `ecnl_legacy` cal value (estimated: Boys 80–90, Girls 85–95 — midpoint between `ecnl` and `ecnl_rl`). Path A teams keep most of their historical rating momentum; the transition is softer.

**What this costs:**
- A DDL change is required (SENTINEL authorization needed before May 10)
- The `ecnl_legacy` cal value cannot be derived from cross-league win-rate data ELO currently has — ELO would be declaring an estimate, not computing from evidence
- Ongoing maintenance: two separate ECNL tiers in the calibration system indefinitely

**When to prefer Option 3 over Option 2:**
- Option 2 cliff exceeds 100 pts downward for > 20% of Path A teams (measured after the re-tag is run on a test set)
- Presten explicitly prefers the two-track system over the cliff
- DDL authorization is obtained from SENTINEL before May 10

**ELO's condition for switching to Option 3:** If Option 2 impact analysis (run from the CP1-FAIL branch) shows the average Path A team cliff exceeds 80 pts, ELO will recommend Option 3 to SENTINEL and Presten as the lower-distortion path.

---

## Section 5: May 10 Deadline Implications

If CP1 FAILS on April 29, the calendar is:

| Date | Event | Required Action |
|------|-------|----------------|
| **April 29** | CP1 FAIL confirmed | ELO files Queue alert, flags briefing, updates Option Comparison Matrix. This is Step 1. |
| **April 29 – May 2** | SENTINEL reads CP1 FAIL alert | SENTINEL schedules Presten session to review Option 2 cliff. **Target: Presten session confirmed by May 2.** |
| **May 3–5** | Presten session: review Option 2/3 | Presten reviews cliff magnitude estimate, selects preferred option, signs off in writing. |
| **May 5** | ELO confirmation deadline | If no Presten sign-off by May 5: ELO defaults to Option 3 (ecnl_legacy) as the safer non-cliff fallback and files this in the Queue as a decision-by-default. SENTINEL is notified. |
| **May 5–9** | ELO prepares migration spec for selected option | Implementation spec for Option 2 (mass UPDATE query + recompute plan) or Option 3 (DDL + assignment UPDATE) written and ready for FORGE. |
| **May 10** | Option selection deadline | ELO files CP7 selection in `Rankings/ECNL Migration — Option Selection Decision Gate.md`. Option selected by this date regardless of whether Presten sign-off arrived — default to Option 3 if not. |
| **June 1** | Migration execution | FORGE runs the selected option's implementation. |

**Critical path:** The 13-day window between April 29 and May 10 is tight. If SENTINEL does not move within 3 days of CP1 FAIL, the Presten session window compresses to < 5 days. The May 5 default is a real default — ELO will select Option 3 without Presten sign-off if nothing is confirmed by then.

---

## References

- `Rankings/ECNL Migration — Option Comparison Matrix.md` — Running Verdict Tracker to update (Section 4)
- `Rankings/April 29 ELO Session Master Script.md` — Step 1 context; CP1 is the first action of the session
- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` — Option 2/3 technical definitions and calibration delta analysis
- `Rankings/ECNL Migration — Option Selection Decision Gate.md` — final decision record; CP7 row receives the selected option
- `task-2026-04-26-ecnl-migration-option-comparison-matrix.md` — Option Comparison Matrix source task

*ELO — 2026-04-26*
