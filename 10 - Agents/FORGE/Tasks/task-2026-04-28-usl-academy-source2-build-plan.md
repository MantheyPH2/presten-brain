---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: pending
priority: high
due: 2026-05-06
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source #2 — USL Academy Implementation Build Plan.md"
---

# Task: Non-GotSport Source #2 — USL Academy Implementation Build Plan

## Context

FORGE filed the Source #1 (EDP) implementation build plan today. EDP and USL Academy share the same activation path — both are GotSport-platform sources accessible via org-ID config add, not new parser development. The EDP build plan compresses to 4 days because of this.

USL Academy is Source #2 in the non-GotSport priority list. Its activation gates on EDP Gate 1 (parse quality) + Gate 2 (ranking impact), which are expected ~May 4-6. When those gates pass, Presten should be able to activate USL Academy in the same session without a planning delay.

The gap: no USL Academy-specific build plan exists. The EDP build plan references USL Academy as "follows EDP in same browser session" but does not specify: which org-IDs to add, what game volume to expect, how to validate the ingestion, or what the ranking impact profile looks like.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Source #2 — USL Academy Implementation Build Plan.md`

Model after the EDP build plan. USL Academy's path is simpler in some ways (same parser, same config-add mechanic) but has a unique consideration: USL Academy includes both elite boys and girls teams, so ELO validation of ranking impact will focus on the boys side (where calibration is currently under active review).

---

### Section 1: Source Identity and Summary

| Field | Value |
|-------|-------|
| Source name | USL Academy |
| Platform / data format | GotSport API |
| Parser reuse | YES — same parser as EDP and existing GotSport sources |
| Org-ID(s) required | [FORGE documents from USL Academy org-ID research] |
| Game volume estimate | [from source priority list and prior research] |
| Pre-DSS Safe | YES (activates after EDP Gate 1 + Gate 2, estimated May 4-6) |
| Implementation window | EDP Gate 2 pass → same-session config add |
| ELO coordination required | YES — boys rating impact flag if >50 pts shift |

---

### Section 2: Org-ID Inventory

FORGE documents all USL Academy org-IDs to be added:

| Org-ID | Verified? | Source of Verification |
|--------|-----------|------------------------|
| [id 1] | YES / NO | [browser session / prior research / other] |
| [id 2] | YES / NO | |
| ... | | |

Cross-reference: `task-2026-04-27-usl-academy-org-id-research.md` — use that document's confirmed org-IDs. Flag any that were NOT confirmed in the browser session.

---

### Section 3: Day-by-Day Build Timeline

USL Academy activates in the same session as EDP Gate 2 confirmation. Compress timeline accordingly.

| Step | Expected Date | Task | Owner | Success Criteria |
|------|--------------|------|-------|-----------------|
| 0 | EDP Gate 2 pass day (~May 4-6) | Add USL Academy org-IDs to config | FORGE | Config syntax valid |
| 0 | Same session | Run USL Academy initial crawl | Presten | No parse errors |
| 1 | Day +1 | First game count verification | FORGE | Games present in DB, correct schema |
| 2 | Day +2 | ELO ranking impact check | ELO | No anomalous boys rating shifts > 50 pts |
| 2 | Day +2 | FORGE validation report filed | FORGE | Filed at `Infrastructure/Source #2 — First Run Validation.md` |
| 3 | Day +3 | SENTINEL authorization | SENTINEL | Gate 3 signed |

---

### Section 4: ELO Coordination Requirements

USL Academy data may affect boys rankings, where Option A calibration changes are in active evaluation (April 29–May 5 window). FORGE must flag this to ELO before the USL Academy crawl runs.

Specific coordination:
1. FORGE notifies ELO: "USL Academy adding — run post-ingestion boys ranking comparison"
2. ELO threshold: any boys team moving >50 pts from USL Academy data alone = anomaly flag
3. ELO files confirmation within 24 hours of ingestion: "USL Academy data is calibration-safe for boys ratings"
4. If ELO flags anomaly: FORGE disables USL Academy in config, files failure report, waits for SENTINEL direction

---

### Section 5: Validation Gates

**Gate 1 — Parse Quality**
- Parse error rate < 5%
- All required fields present: team_name, opponent_name, date, score, league_name
- No duplicate game rows (same teams + date)
- Expected game count within ±30% of estimate from Section 1

**Gate 2 — Ranking Impact (ELO)**
- ELO runs post-ingestion boys ranking comparison
- No boys team moves > 50 pts from USL Academy data alone
- Girls ranking impact acceptable (< 30 pts avg shift)
- ELO files confirmation: "Source #2 data is calibration-safe"

**Gate 3 — SENTINEL Authorization**
- FORGE files first-run validation report at `Infrastructure/Source #2 — First Run Validation.md`
- SENTINEL reviews and issues production authorization
- Presten confirms USL Academy is live in daily pipeline

---

### Section 6: Rollback Plan

If Gate 1 or Gate 2 fails:
- Disable USL Academy org-IDs in config (leave EDP active)
- FORGE files `Infrastructure/Source #2 — First Run Failure Report.md` within 2 hours
- SENTINEL decides: revise parser or config, retry, or defer post-DSS
- ELO is notified immediately if boys ranking impact is the failure cause

---

## Definition of Done

- Build plan filed before EDP Gate 1 completes (~May 4)
- Section 2 org-ID inventory populated from prior USL Academy research
- Section 3 has specific calendar dates (not relative steps)
- ELO coordination protocol (Section 4) documented and ELO acknowledged
- Validation gates have specific numeric thresholds

## Why This Matters

If USL Academy's build plan doesn't exist before EDP Gate 2 passes, Presten faces an exploratory session instead of execution. With DSS on May 16 and boys calibration in active evaluation, the window for USL Academy is tight. The plan must be ready before it's needed.

## References

- `Infrastructure/Non-GotSport Source #1 — Implementation Build Plan.md` — EDP; use as structural template
- `task-2026-04-27-usl-academy-org-id-research.md` — org-ID confirmation (input to Section 2)
- `Infrastructure/Non-GotSport Source Implementation Priority List.md` — confirms USL Academy = Source #2
- `task-2026-04-28-edp-usl-academy-config-activation-card.md` — activation card (already filed)
- `task-2026-04-27-boys-option-a-change-execution-protocol.md` — boys calibration context for ELO gate
