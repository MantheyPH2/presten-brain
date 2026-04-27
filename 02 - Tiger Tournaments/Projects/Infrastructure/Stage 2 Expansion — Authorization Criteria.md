---
title: Stage 2 Expansion — Authorization Criteria
tags: [infrastructure, gotsport, stage2, authorization, org-ids, pipeline, forge]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: awaiting SENTINEL pre-approval (Section 5)
task: task-2026-04-27-stage2-authorization-criteria-doc
---

# Stage 2 Expansion — Authorization Criteria

> **Purpose:** Pre-defined thresholds and conditions that constitute a Stage 2 GO decision. Written before Stage 1 runs so that the eventual Stage 2 authorization request is a pass/fail check against known criteria, not a new judgment call. SENTINEL pre-approves these criteria in Section 5; FORGE files the authorization request when all gates are met.

---

## Section 1: Stage 1 Completion Gate

Stage 2 cannot begin until Stage 1 TX validation is complete. Stage 1 is the single-entity TYSA pilot crawl.

### Required (all must be met)

- Stage 1 Post-Run Validation Checklist (`Infrastructure/Stage 1 TX Crawl — Post-Run Validation Checklist.md`) executed in full — all 4 validation queries (V1–V4) run and results recorded.
- **V1 result (new game count by org-ID): GREEN** — ≥ 5,000 new games ingested from TYSA org-ID. Source: Stage 1 Expected Output Spec GREEN floor.
- **V3 result (duplicate game check): 0 rows** — no duplicate game entries for Stage 1 TX org-IDs.
- **V4 result (date range sanity):** Games present with `game_date` in the 2025–2026 season window (at least some games between 2025-08-01 and 2026-07-31).

### Conditional (may proceed with SENTINEL review)

- **V1 result YELLOW (500–4,999 games):** FORGE files a written explanation with SENTINEL identifying which TYSA sub-events were and were not accessible before Stage 2 authorization is considered. SENTINEL may authorize a partial Stage 2 (one additional state rather than the full VA/CO/AZ scope) pending Stage 1 rerun or scope clarification.
- **V2 result (team match rate) < 80%:** FORGE files a dedup analysis explaining why match rate is low (new TX teams not in DB, name format mismatch, etc.) before Stage 2 authorization. Stage 2 expansion will bring new teams — if Stage 1 match rate is already weak, Stage 2 may compound dedup issues. SENTINEL decides whether to proceed or require match-rate improvement first.

### Hard Stop

- **V1 result RED (< 500 games, or 0 games from TYSA org-ID):** Stage 2 blocked until Stage 1 is diagnosed and rerun with a GREEN result. FORGE does not file a Stage 2 authorization request under this condition.
- **V3 result: any duplicates present (> 0 rows):** Stage 2 blocked until Stage 1 dedup issue is diagnosed and resolved. Duplicates at Stage 1 scale indicate a pipeline dedup failure that will compound at Stage 2 volume.

---

## Section 2: May 1 Pipeline Health Gate

Stage 2 adds new org-IDs to the production daily pipeline. The pipeline must be confirmed healthy — running and producing data — before new sources are added.

### Required (all must be met)

- May 1 first-run validation checklist (`Infrastructure/May 1 First-Run Validation Checklist.md`) filed with overall verdict of GREEN or acceptable YELLOW.
- 4-day game count (May 1–4, from `Infrastructure/May 2–4 Pipeline Monitoring Checklist.md`): **≥ 700 games** (GREEN threshold per Section 3 of that document).
- No active RED signals as of Stage 2 authorization request date: no unresolved cron failure, no DB write-path failure, no pipeline exception producing 0 games across consecutive days.

### Conditional

- **YELLOW signals on May 3–4 that have not resolved to GREEN by May 5:** FORGE must include the full pipeline health trend data (3–4 day comparison table from the monitoring checklist) in the Stage 2 authorization request. SENTINEL may delay Stage 2 authorization by 1 week (to May 12) to confirm pipeline health stabilizes.
- **One source RED, others GREEN (source-specific failure, not systemic):** FORGE files the source-specific issue and diagnosis with the Stage 2 authorization request. SENTINEL decides whether the isolated source failure blocks Stage 2 or is acceptable given the other sources are healthy.

### Hard Stop

- **Pipeline aggregate RED after 48 hours (< 50 games May 1–2):** Stage 2 blocked until pipeline is diagnosed and GREEN for ≥ 3 consecutive days. A pipeline not reliably running at its current Stage 1 scope cannot absorb Stage 2 org-ID additions.
- **DB write-path failure confirmed (pipeline runs but 0 new games in DB):** Stage 2 blocked until write-path fix is deployed, confirmed, and pipeline produces GREEN results for ≥ 2 consecutive days.

---

## Section 3: Org-ID Confirmation Gate

Stage 2 expansion requires confirmed GotSport org-IDs for entities that FORGE intends to add. As of April 27, the current Stage 2 pending entity list is the 11 browser-session entities plus the 3 Stage 3 candidates (VA, CO, AZ). The minimum set for Stage 2 activation:

**Stage 2 minimum scope:**
At least 3 of the 11 browser-session entities confirmed with active org-IDs. These are the USYS state associations (Cal North, Cal South, FYSA, ENYYSA, NJYSA, WSYSA, EPYSA) and national leagues (USL Academy, EDP Soccer, Elite 64, NAL). Any 3 confirmed entities make Stage 2 viable; the full 11 is the target.

### Required

- **Browser session completed:** Presten has run the system.gotsport.com lookup and reported results to FORGE.
- **At least 3 Stage 2 entities confirmed with numeric org-IDs** and added to `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` with status `confirmed`.
- **Stage 2 Pre-Staged Config** (`Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md`): `ORG_ID_PENDING` placeholders replaced with confirmed numeric values for all confirmed entities.

### Conditional

- **If only 1–2 entities confirmed (partial browser session):** FORGE may not file a Stage 2 authorization request. Minimum of 3 confirmed org-IDs is required for Stage 2 to be worth the deployment cycle. FORGE informs SENTINEL that Stage 2 is pending more browser session results.
- **If VA/CO/AZ (Stage 3) entities have confirmed org-IDs but Stage 2 entities do not:** VA/CO/AZ are Stage 3 candidates. They do not count toward the Stage 2 minimum of 3 confirmed entities.
- **If 3–10 of 11 entities confirmed:** Proceed with partial Stage 2 (confirmed entities only). Remaining entities are deferred to Stage 2b or Stage 3 when browser session resumes.

### Hard Stop

- **No browser session completed at all:** Stage 2 blocked. FORGE cannot self-generate org-IDs. Without browser confirmation, the config entries are placeholders only and cannot be deployed.

---

## Section 4: Authorization Request Format

When all three gates are met (or conditional criteria are acceptable), FORGE files the Stage 2 authorization request in the FORGE briefing:

```
**To:** SENTINEL
**From:** FORGE
**Re:** Stage 2 GotSport Expansion — Authorization Request

**Stage 1 Gate:**
  Verdict: [PASS / CONDITIONAL — explain]
  V1 (new game count): [X games] — [GREEN / YELLOW / RED]
  V2 (team match rate): [X%] — [OK / FLAG]
  V3 (duplicates): [0 / X rows] — [OK / FLAG]
  V4 (date range): [confirmed in 2025–26 / FLAG — describe]

**May 1 Pipeline Gate:**
  Verdict: [PASS / CONDITIONAL — explain]
  4-day aggregate (May 1–4): [X games] — [GREEN / YELLOW / RED]
  Active RED signals: [none / describe]
  gotsport_api status: [GREEN / YELLOW / RED]
  tgs_athleteone status: [GREEN / YELLOW / RED]

**Org-ID Gate:**
  Browser session completed: [YES / PARTIAL — N of 11 entities resolved]
  Confirmed entities: [list — entity name + org-ID]
  Not-on-GotSport: [list — entity name + reason]
  Stage 2 config staged and ready: [YES / PARTIAL]

**FORGE Recommendation:**
  [ ] PROCEED — full Stage 2 (all [N] confirmed entities)
  [ ] PROCEED — partial Stage 2 ([specific entities]; [N] entities pending)
  [ ] HOLD — [explain which gate is not met and what is needed]

**Estimated Stage 2 game volume:**
  [X–Y games, based on confirmed entity types and known league game volume from VA/CO/AZ research and state association baselines]

Requesting SENTINEL authorization to add Stage 2 org-IDs to production crawl config.
```

**Turnaround expectation:** FORGE files this request when all gates are met. SENTINEL reviews and responds within the same session. If SENTINEL does not respond within 24 hours, FORGE re-files the request in the next briefing.

---

## Section 5: SENTINEL Pre-Approval of These Criteria

*This section is reserved for SENTINEL to fill. If SENTINEL approves the criteria as written, SENTINEL adds a disposition here. If any criteria require modification, SENTINEL notes the change below.*

```
SENTINEL Disposition:

[ ] Criteria reviewed and approved as written.
    Date: ___________
    Note: "FORGE may file Stage 2 authorization request against these criteria
          without a new SENTINEL review of the criteria themselves."

[ ] Criteria modified. Changes:
    ___________
    Date approved: ___________
```

---

## Criteria Version Log

| Version | Date | Change | Author |
|---------|------|--------|--------|
| 1.0 | 2026-04-27 | Initial write — all three gates defined | FORGE |

---

## References

- `Infrastructure/Stage 2 Expansion — Authorization Trigger Document.md` — the document FORGE fills in when gates are met; this criteria doc defines what "met" means
- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — defines Stage 1 GREEN/YELLOW/RED thresholds (Section 1 of this doc)
- `Infrastructure/Stage 1 TX Crawl — Post-Run Validation Checklist.md` — Stage 1 gate execution
- `Infrastructure/May 1 First-Run Validation Checklist.md` — May 1 pipeline gate
- `Infrastructure/May 2–4 Pipeline Monitoring Checklist.md` — 4-day aggregate threshold (Section 2)
- `Infrastructure/May 1 — Pipeline Red Signal Response Protocol.md` — defines RED signal conditions (Section 2 hard stop)
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — org-ID confirmation status (Section 3)
- `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — config ready for Section 3 deployment

*FORGE — 2026-04-27*
