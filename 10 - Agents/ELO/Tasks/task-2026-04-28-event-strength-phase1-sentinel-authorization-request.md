---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: pending
priority: high
due: 2026-04-30
trigger: G0 = GO confirmed
deliverable: "10 - Agents/SENTINEL/Queue/pending-2026-04-29-event-strength-phase1-authorization.md"
---

# Task: Event Strength Phase 1 — SENTINEL Authorization Request

## Context

Event Strength Phase 1 is blocked on two conditions: (1) G0 = GO, (2) SENTINEL authorization. G0 resolves tonight or April 29. The SENTINEL authorization request is the document ELO files to unblock Phase 1 — and it does not exist yet.

The problem: ELO has the Phase 1 execution package (`task-2026-04-27-event-strength-phase1-elo-execution-package.md`), the authorization criteria (`task-2026-04-25-event-strength-phase1-authorization-criteria.md`), and the evidence package (`task-2026-04-26-event-strength-phase2-authorization-evidence-package.md`). But none of those are addressed to SENTINEL in queue format. SENTINEL cannot authorize what it hasn't been formally asked to authorize.

This task produces the formal SENTINEL authorization request — the specific document ELO files to `10 - Agents/SENTINEL/Queue/` after G0 = GO is confirmed.

## What to Produce

File: `10 - Agents/SENTINEL/Queue/pending-2026-04-29-event-strength-phase1-authorization.md`

File within 24 hours of G0 = GO confirmation. Do not file before G0 is confirmed.

---

### Section 1: Request Header

```yaml
---
type: agent-queue
agent: ELO
category: recommendation
priority: high
date: 2026-04-29
status: pending
answer: ""
---
```

---

### Section 2: Authorization Request Summary

**Requesting:** SENTINEL authorization to activate Event Strength Phase 1 in production.

**What Event Strength Phase 1 does:** Recalculates event weight in the ELO model based on verified event tier data. Phase 1 covers girls ratings only. It does not change the calibration algorithm — it changes the input weight of games from different tiers. Expected effect: elite-tier games carry more weight; lower-tier games carry less. Net effect on ratings: moderate (< 30 pts avg shift across affected teams).

**Why now:** G0 = GO means April 28 session ran cleanly. GA ASPIRE fix is in production. Girls calibration baseline is stable. Phase 1 can activate without contaminating the April 28 fix validation.

---

### Section 3: Evidence Summary

ELO documents the current evidence status against each Phase 1 authorization criterion:

| Criterion | Status | Evidence Location |
|-----------|--------|-------------------|
| Girls calibration stable (post-April-28 fix) | [PASS / PENDING] | `Rankings/April 29 Gate Results.md` — G1 result |
| Event tier data coverage sufficient (≥ 80% of games have verified tier) | [PASS / PENDING] | `task-2026-04-25-event-strength-phase1-data-coverage-check.md` |
| Phase 1 expected rating shift < 30 pts avg | [PASS / PENDING] | `task-2026-04-25-event-strength-phase1-rankings-impact-preview.md` |
| No active boys calibration changes in flight | [PASS / PENDING] | Boys Option A verdict (April 29) |
| FORGE pipeline stable (Week 1 GREEN) | [PENDING] | `Infrastructure/Pipeline Week 1 — Game Volume Actuals Report.md` |

ELO fills the Status column after April 29 session results are in.

---

### Section 4: Risk Assessment

**Risk 1: Unexpected rating shift magnitude**
- Mitigation: Phase 1 is girls-only; boys unaffected. ELO monitors post-activation and files anomaly report within 48 hours if > 50 teams move > 50 pts.

**Risk 2: DSS demo disruption**
- Mitigation: Phase 1 activates ≥ 7 days before May 16 DSS. If activation is after May 9, ELO recommends deferring to post-DSS.
- Authorization requested for activation window: May 1–9 only.

**Risk 3: ECNL migration interference**
- Mitigation: Phase 1 does not touch ECNL-specific event tier logic. ECNL migration (June 2) is independent.

---

### Section 5: What ELO Needs from SENTINEL

**SENTINEL decision required:** AUTHORIZE or DEFER.

If **AUTHORIZE:**
- ELO activates Event Strength Phase 1 in next available session after SENTINEL sign-off
- ELO files post-activation monitoring report within 48 hours of activation
- ELO notifies Presten via next briefing

If **DEFER:**
- ELO holds Phase 1 pending SENTINEL's specified condition for re-evaluation
- Phase 2 authorization remains blocked

**SENTINEL response deadline:** April 30 (to allow May 1 activation window before May 9 DSS preflight).

---

## Definition of Done

- Request filed at `10 - Agents/SENTINEL/Queue/pending-[date]-event-strength-phase1-authorization.md` within 24 hours of G0 = GO
- All Section 3 criteria statuses filled from April 29 session results
- ELO's April 29 or April 30 briefing references this request and states: "Awaiting SENTINEL decision"
- ELO does NOT activate Phase 1 until SENTINEL files a decision

## Why This Matters

Event Strength Phase 1 improves ranking accuracy by correctly weighting elite vs. lower-tier game results. It is on the DSS demo critical path. The authorization request is the formal mechanism SENTINEL needs to review and approve. Without it, Phase 1 stays blocked indefinitely — not because the evidence isn't ready, but because the request was never made.

## References

- `task-2026-04-27-event-strength-phase1-elo-execution-package.md` — execution package (ready)
- `task-2026-04-25-event-strength-phase1-authorization-criteria.md` — criteria checklist
- `task-2026-04-26-event-strength-phase2-authorization-evidence-package.md` — evidence compiled
- `task-2026-04-25-event-strength-phase1-rankings-impact-preview.md` — impact preview
- `task-2026-04-25-event-strength-phase2-prerequisites.md` — Phase 2 gates on Phase 1 SENTINEL sign-off
