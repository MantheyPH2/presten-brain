---
title: May 1 Launch — Go No-Go Checklist
tags: [infrastructure, may1, launch, stage1, sentinel, authorization, checklist]
created: 2026-04-29
updated: 2026-04-29
author: FORGE
status: pre-filled — awaiting SENTINEL authorization (Section 4)
linked_task: task-2026-04-29-may1-launch-go-checklist-final
sentinel_fills: "Section 3 (ELO conditions), Section 4 (Launch Authorization)"
---

# May 1 Launch — Go/No-Go Checklist

> **SENTINEL authorization gate document. SENTINEL signs Section 4 on 2026-04-30 EOD.**
> FORGE pre-filled all FORGE-answerable conditions based on vault documents filed through 2026-04-29.
> OPEN items are explicitly marked. Section 4 is blank and reserved for SENTINEL.

**Prepared by:** FORGE — 2026-04-29
**SENTINEL authorization required by:** 2026-04-30 EOD
**Pipeline launch target:** 2026-05-01

---

## Section 1 — Stage 1 Config Conditions

| Condition | Status | Notes | Source |
|-----------|--------|-------|--------|
| gotsport_api pipeline framework confirmed active | PASS ✅ | Existing crawler (`crawl-gotsport-api-parallel.js`) is active and operational. The pipeline itself is confirmed. TYSA org-ID entry is the only missing component — see row 5. | `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` §§1–3 |
| tgs_athleteone config confirmed active | PASS ✅ | Existing active pipeline. Runs daily ECNL AthleteOne data ingestion. No config changes in Stage 1 scope. | Existing pipeline — no changes in scope for Stage 1 |
| tgs config confirmed active | PASS ✅ | Existing active pipeline. TGS scraper runs on daily cadence. | Existing pipeline |
| tgs_rl config confirmed active | PASS ✅ | Existing active pipeline. ECNL RL source. | Existing pipeline |
| TYSA org-ID received and staged | **OPEN ⏳** | Pending Presten browser session at `system.gotsport.com` → search "Texas Youth Soccer" → extract `?org_id=XXXX`. **This is the only actual launch blocker.** Without the org-ID, Stage 1 config entry cannot be written and the TYSA crawl cannot run. | `Infrastructure/GotSport Browser Lookup Session Prep Package.md`; `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` §1 |
| No Stage 1 config has unresolved syntax errors | PASS ✅ (conditional) | Existing pipeline configs have no known syntax errors. TYSA entry cannot be syntax-validated until org-ID is received. FORGE runs `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"` immediately after TYSA entry is added — must pass before pipeline is triggered. | `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` §2; §4 Step 3 |

---

## Section 2 — Infrastructure Conditions

| Condition | Status | Notes | Source |
|-----------|--------|-------|--------|
| FORGE Confidence Brief filed | PASS ✅ | Filed 2026-04-29 04:29. Medium–High confidence; one unresolved item (TYSA org-ID). | `Infrastructure/May 1 Stage 1 — Launch Confidence Brief.md` |
| Week 1 Monitoring Log ready for live fill | PASS ✅ | Template pre-populated with dates, source names, and baseline thresholds. All data cells are blank and ready to fill on May 1. | `Infrastructure/Pipeline Week 1 Monitoring Log.md` |
| Per-source game volume thresholds set | PASS ✅ | GREEN floor: ≥5,000 games first run. YELLOW: 500–4,999 (investigate before Stage 2). RED: <500 (diagnose before proceeding). Pre-populated in Monitoring Log. | `Infrastructure/May 1 — Per-Source Game Volume Forecast.md`; `Infrastructure/Pipeline Week 1 Monitoring Log.md` |
| FM1 behavior confirmed (or pre-draft staged) | PRE-DRAFT STAGED ⚠️ | FM1 Audit Pre-Draft filed 2026-04-29. Awaiting Presten Pipeline Code Review Results (grep commands ready at `Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md`). **Not a May 1 launch blocker** — GA ASPIRE classification is an accuracy concern, not a launch gating requirement. | `Infrastructure/FM1 Audit — Pre-Draft.md` |
| FM2 behavior confirmed (or pre-draft staged) | PRE-DRAFT STAGED ⚠️ | FM2 Audit Pre-Draft filed 2026-04-29. Awaiting Presten Pipeline Code Review Results. FM2 Path B fix (ecnl_verified written at ingest time) preferred before May 1. If FM2 = Path C: Stage 2 authorization held until fix deployed. **Not a Stage 1 launch blocker** but affects ecnl_verified accuracy for all post-May-1 ECNL games. | `Infrastructure/FM2 Audit — Pre-Draft.md` |
| Archive dry-run authorization status | OPEN ⏳ | Pending Presten dry-run execution (5-min bash command). **NOT a launch blocker** per FORGE Confidence Brief. Archive workflow is entirely separate from Stage 1 TYSA crawl pipeline. | `Infrastructure/Archive Inactive Teams — Production Execution Authorization Request.md` |

---

## Section 3 — ELO Calibration Conditions

*SENTINEL fills this section on 2026-04-30 EOD based on ELO's April 29 session output.*

| Condition | Status | Source |
|-----------|--------|--------|
| ELO R5 stddev baseline filed before May 1 | [SENTINEL fills on April 30] | ELO task-2026-04-29-r5-stddev-baseline-pre-staging |
| Girls GA ASPIRE fix status | [SENTINEL fills on April 30] | April 29 session status |
| ELO confirms pipeline launch does not destabilize rankings | [SENTINEL fills on April 30] | ELO May 1 pipeline launch monitoring plan |

---

## Section 4 — Launch Authorization

*SENTINEL fills this section on 2026-04-30 EOD. Do not pre-fill.*

```
LAUNCH DECISION: [ ] GO  [ ] NO-GO  [ ] CONDITIONAL GO

Conditions assessed: ___/10

Open conditions at launch:
- TYSA org-ID: [ ] resolved  [ ] still open
- FM1 audit: [ ] completed  [ ] pre-draft only
- FM2 audit: [ ] completed  [ ] pre-draft only
- ELO R5 baseline: [ ] filed  [ ] not filed
- Archive dry-run: [ ] run  [ ] not run (not a blocker per Confidence Brief)

If NO-GO or CONDITIONAL GO — specific items requiring resolution before launch:
1.
2.

SENTINEL signature: _________________________ Date: 2026-04-30
```

---

## Quick Reference — Blockers vs. Non-Blockers

| Item | Blocks May 1 Launch? | Reason |
|------|---------------------|--------|
| TYSA org-ID missing | **YES** | Stage 1 = TYSA only. No org-ID = no config entry = no Stage 1 crawl. |
| FM1 audit incomplete | NO | GA ASPIRE classification is an accuracy concern, not a launch gate. |
| FM2 audit incomplete | NO (if Path A or B) | Pipeline launches; ecnl_verified accuracy degraded for new ECNL games if Path B fix not deployed. If FM2 = Path C: Stage 2 authorization held. |
| Archive dry-run not run | NO | Completely separate workflow with no interaction with Stage 1 TYSA crawl. |
| ELO R5 baseline not filed | SENTINEL judgment | ELO monitoring concern; SENTINEL determines impact on authorization. |
| G0 = NO-GO (ECNL gate) | NO | Stage 1 is ECNL-independent. TYSA crawl proceeds regardless of G0 result. Confirmed in `Infrastructure/April 29 — FORGE G0-Contingent Action Card.md`. |

---

## References

- `Infrastructure/May 1 Stage 1 — Launch Confidence Brief.md` — FORGE readiness narrative and confidence level
- `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` — Stage 1 org-ID manifest, activation commands
- `Infrastructure/FM1 Audit — Pre-Draft.md` — GA ASPIRE classifier audit (fill on receipt of Presten code review)
- `Infrastructure/FM2 Audit — Pre-Draft.md` — ecnl_verified write path audit (fill on receipt of Presten code review)
- `Infrastructure/Pipeline Week 1 Monitoring Log.md` — first-week tracking template
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — full session execution guide
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — Stage 2 gate (requires "Stage 1 confirmed successful")

*FORGE — 2026-04-29*
