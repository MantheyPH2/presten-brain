---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: open
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md"
---

# Task: May 1 Stage 1 — Config Final Review and Activation Card

## Context

May 1 is the pipeline deployment date. Stage 1 activates the first wave of new GotSport org IDs (TX expansion). FORGE has produced individual research documents (org ID lists, expected output specs, day-of runbooks, validation checklists, monitoring queries). What does not exist: a single reviewed, consolidated activation card that Presten can use during the May 1 session to confirm the config is correct before starting the pipeline.

The risk: Presten activates Stage 1 on May 1 with a config file that contains a typo, a duplicate org ID, or an inactive org ID that was identified as such in later research but not propagated back to the config. FORGE needs to do one final review of the planned config against all research, confirm there are no contradictions, and produce a one-page activation card that serves as FORGE's signed certification that the config is deployment-ready.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md`

---

### Section 1: Stage 1 Org-ID Manifest

List every org ID planned for Stage 1 activation. For each:

| Org ID | Club / Organization | State | League | Status | Source Doc |
|--------|-------------------|-------|--------|--------|------------|
| | | | | Confirmed / Unverified / Needs Browser Check | |
| | | | | | |

Status definitions:
- **Confirmed** = org ID verified as active via browser lookup or authenticated FORGE research
- **Unverified** = org ID identified via research but not browser-confirmed; flagged for Stage 2 verification
- **Needs Browser Check** = FORGE recommends holding this org ID until Presten completes browser session

FORGE includes only **Confirmed** org IDs in the Stage 1 activation set. Any Unverified or Needs Browser Check org IDs are staged for post-browser-session activation.

---

### Section 2: Config Contradiction Check

FORGE reviews all org ID research documents filed since April 22 and confirms no contradictions:

- [ ] No org ID appears in both the active list and the inactive/flagged list
- [ ] No duplicate org IDs in the Stage 1 set
- [ ] TYSA org IDs: mark status (Confirmed / Needs Browser Check — TYSA session pending)
- [ ] No org IDs flagged as "wrong state" or "wrong club" in the research documents
- [ ] VA / CO / AZ org IDs: mark status (from `task-2026-04-27-va-co-az-gotsport-org-id-research.md`)
- [ ] Pacific Northwest org IDs: mark status (from `task-2026-04-27-pacific-northwest-gotsport-org-id-research.md`)

Contradiction log: _[list any discrepancies found, or "none"]_

---

### Section 3: Expected Game Volume

Based on Stage 1 org IDs:

| State | Org IDs Activating | Expected Game Volume (first 7 days) | Calibration League? |
|-------|-------------------|-------------------------------------|---------------------|
| TX | | | |
| Other | | | |
| **Total** | | | |

This table feeds directly into ELO's May 1 Rankings Monitoring Plan. FORGE confirms the volume estimate matches the `task-2026-04-27-may1-per-source-game-volume-forecast.md` figures.

---

### Section 4: Activation Command

Exact command(s) Presten will execute on May 1 to activate Stage 1 org IDs. FORGE provides:

```bash
# [insert exact commands here — no blanks]
```

Pre-activation check:
```sql
-- Run this BEFORE activation to confirm baseline game count
SELECT COUNT(*) FROM games WHERE ingested_at >= NOW() - INTERVAL '7 days';
```

Post-activation check (run within 15 minutes of first pipeline run):
```sql
-- Run this AFTER first Stage 1 pipeline run
SELECT org_id, COUNT(*) as games_ingested
FROM games
WHERE ingested_at >= '[activation timestamp]'
GROUP BY org_id
ORDER BY games_ingested DESC;
```

---

### Section 5: FORGE Certification

> FORGE certifies that the Stage 1 org ID configuration listed in Section 1 has been reviewed against all research documents filed through 2026-04-28. All Confirmed org IDs are verified active. No contradictions were found. The configuration is deployment-ready for May 1.

FORGE certification date: 2026-04-30
Signed: FORGE

SENTINEL review: _PENDING_

---

### Section 6: Known Gaps and Post-May-1 Actions

| Gap | Reason Not in Stage 1 | Next Action |
|-----|----------------------|-------------|
| TYSA org IDs | Pending Presten browser session | Add in Stage 2 after browser confirmation |
| _[other]_ | | |

---

## Definition of Done

- Document filed at deliverable path by April 30
- Section 1 manifest is complete with no blank rows
- Section 2 contradiction check has all boxes checked and contradiction log filled
- Section 4 exact commands are present (no placeholder text)
- FORGE briefing April 30 states: "Stage 1 config is certified deployment-ready"
- SENTINEL reviews and signs off before May 1

## Why This Matters

This is the last review gate before Stage 1 activates. If an incorrect org ID enters production, it produces bad game data that corrupts ELO ratings for an entire club's teams — and the corruption compounds daily until caught. A 30-minute FORGE config audit before May 1 costs nothing. Discovering a bad org ID after 7 days of production data costs 7 days of re-calibration.

## References

- `task-2026-04-27-gotsport-org-id-master-reference.md` — master org ID reference
- `task-2026-04-27-stage2-org-id-config-pre-staging.md` — Stage 2 pre-staging (informs what's NOT Stage 1)
- `task-2026-04-27-va-co-az-gotsport-org-id-research.md` — VA/CO/AZ research
- `task-2026-04-27-pacific-northwest-gotsport-org-id-research.md` — Pacific NW research
- `task-2026-04-27-may1-deployment-day-of-runbook.md` — deployment day runbook
- `task-2026-04-24-full-gotsport-org-id-list.md` — baseline org ID list
