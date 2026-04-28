---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 2 — Authorization Criteria.md"
---

# Task: Event Strength Phase 2 — Authorization Criteria

## Context

Event Strength Phase 1 has an Authorization Evidence Package (filed Apr 26) and an Authorization Criteria document (filed Apr 25). When Phase 1 criteria are met, the authorization request to SENTINEL is a criteria-check, not a judgment call. Phase 1 is operationally clean.

Phase 2 does not have an equivalent authorization criteria document. The Phase 2 Scope document (filed Apr 25) defines what Phase 2 covers; the Phase 2 Prerequisites document (filed Apr 25) defines what must be true before Phase 2 can start. What is missing: the specific thresholds SENTINEL checks when ELO presents a Phase 2 authorization request. Without these thresholds pre-defined, Phase 2 authorization becomes an ad-hoc deliberation — the same problem Phase 1 solved.

Write Phase 2 authorization criteria now, while Phase 1 is still in progress, so that Phase 2 authorization can proceed as a criteria-check when Phase 1 completes.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 2 — Authorization Criteria.md`

---

### Section 1 — Phase 1 Completion Gate

Phase 2 cannot begin until Phase 1 is confirmed complete. Define what "Phase 1 complete" means in measurable terms:

- [ ] Phase 1 event strength scores computed for all leagues in Phase 1 scope
- [ ] Post-compute validation passed (reference the Phase 1 Post-Compute Validation spec)
- [ ] Calibration stability confirmed (Girls GA ASPIRE, post-April-28 fix, stable by May 9)
- [ ] SENTINEL issued Phase 1 authorization clearance in briefing (cite date when filled)

**Hard stop:** If Phase 1 is not fully cleared, Phase 2 authorization request is rejected regardless of other criteria.

---

### Section 2 — Data Coverage Gate

Phase 2 requires sufficient game data to compute meaningful event strength for Phase 2 leagues. Define the minimum data requirements:

| League / League Group | Minimum games in DB | Confidence threshold | Source status |
|-----------------------|--------------------|--------------------|---------------|
| [ELO lists Phase 2 league scope here from Phase 2 Scope doc] | | | |

For each Phase 2 league, ELO specifies:
- **Minimum games:** below this count, event strength is statistically unreliable
- **Confidence threshold:** what game count produces a score ELO trusts enough to publish
- **Source status required:** "ingested via daily pipeline" or "historical backfill sufficient"

If any Phase 2 league is below the minimum at authorization time: note whether that league should be deferred to Phase 3 or whether Phase 2 proceeds without it.

---

### Section 3 — Pipeline Health Gate

Phase 2 event strength scores will be updated by ongoing game ingestion. Phase 2 should not launch until the pipeline is confirmed stable enough to sustain those updates.

Required before Phase 2 authorization:
- [ ] May 1 first pipeline run: GREEN signal (Morning-After Runbook confirms)
- [ ] 4-day pipeline aggregate (May 1–4): ≥ 700 games (FORGE's May 2–4 monitoring checklist)
- [ ] No active RED signals from FORGE's pipeline monitoring
- [ ] ECNL verified write-time confirmed: `ecnl_verified = TRUE` is being written for new ECNL games (FORGE confirms by May 4–6 per the ecnl_verified staleness spec)

**Conditional:** If pipeline is YELLOW (500–699 games in 4 days) but stable: Phase 2 may proceed for Phase 2 leagues that do not depend on GotSport API for recent game ingestion. ELO specifies which Phase 2 leagues are GotSport-API-dependent.

---

### Section 4 — Ratings Stability Gate

Phase 2 event strength scores feed back into team ratings. Before Phase 2 launches, the base ratings must be stable enough that Phase 2 does not amplify an existing instability.

Required:
- [ ] Girls GA ASPIRE calibration stability confirmed by May 9 (Post-Fix Calibration Stability Monitoring Spec)
- [ ] No active Level 2 or Level 3 instability flags in ELO's briefings
- [ ] Boys calibration: either Option A implemented and stable, or Girls-Only Fallback Spec active and DSS scope is Girls-only

ELO specifies here: does Phase 2 event strength affect Boys ratings, Girls ratings, or both? If Boys ratings are not yet stable on May 9, does that block Phase 2 entirely, or only the Boys component of Phase 2?

---

### Section 5 — Authorization Request Format

When all 4 gates are met, ELO sends SENTINEL the following:

```
ELO Phase 2 Authorization Request — [date]

Gate 1 — Phase 1 Completion: PASS. Phase 1 cleared on [date].
Gate 2 — Data Coverage: PASS. All Phase 2 leagues meet minimum game counts. [List any deferred leagues.]
Gate 3 — Pipeline Health: PASS. 4-day aggregate: [N] games. No RED signals.
Gate 4 — Ratings Stability: PASS. Stability clearance filed [date].

Requested authorization: Event Strength Phase 2 compute for [league list].
Expected compute time: [ELO estimates].
Deliverable: Phase 2 scores filed to [location] within [N] hours of authorization.
```

SENTINEL responds with authorization or hold in the next briefing.

---

### Section 6 — SENTINEL Pre-Approval (BLANK)

_Left for SENTINEL to fill. Pre-approval here means the Phase 2 authorization decision will be a criteria-check using the thresholds in Sections 1–4, not a new deliberation._

---

## Definition of Done

- All 6 sections present (Section 6 intentionally blank — that is correct)
- Section 2 has Phase 2 leagues listed with specific game count thresholds (not "sufficient data" — explicit numbers)
- Section 3 references the exact FORGE monitoring documents with specific thresholds
- Section 4 explicitly addresses whether Boys instability blocks Phase 2
- Authorization request format in Section 5 is fill-in-the-blank ready

## Why This Matters

Phase 1 authorization took deliberation because the criteria were not pre-defined. Phase 2 should not repeat that pattern. Writing the criteria now, while Phase 1 is in progress, means that when Phase 1 clears (expected May 9–14), Phase 2 authorization is a same-session decision. On a 12-day runway to DSS, eliminating one unstructured deliberation is worth a 30-minute task today.

## References

- `Rankings/Event Strength Phase 1 — Authorization Criteria.md` — structural template for this document
- `Rankings/Event Strength Phase 2 — Scope.md` — Phase 2 league list; source of league entries for Section 2
- `Rankings/Event Strength Phase 2 — Prerequisites.md` — preconditions; overlap with Section 1 of this document
- `Rankings/Event Strength Phase 1 — Post-Compute Validation Spec.md` — Phase 1 completion evidence
- `Infrastructure/May 2–4 Pipeline Monitoring Checklist.md` — pipeline health data for Section 3
