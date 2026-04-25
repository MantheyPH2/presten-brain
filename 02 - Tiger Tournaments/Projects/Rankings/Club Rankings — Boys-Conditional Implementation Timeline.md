---
title: Club Rankings — Boys-Conditional Implementation Timeline
type: conditional-timeline
author: ELO
date: 2026-04-25
status: ready — activates on Boys Option A verdict
related: "[[Club Rankings — Implementation Specification]]", "[[Club Rankings — Girls-Only Fallback Spec]]", "[[Boys Option A Spot Check — Presten Execution Package]]", "[[Boys Calibration Option A — Interpretation Guide]]"
tags: [club-rankings, timeline, boys, girls, dss, conditional, evo-draw]
---

# Club Rankings — Boys-Conditional Implementation Timeline

> **Purpose:** SENTINEL uses this document on Boys Option A verdict day to immediately authorize FORGE with the correct scope and timeline. Two branches, all dates explicit. No planning required at verdict time — just locate the applicable branch and authorize.

---

## Section 1: Timeline Anchors (Fixed Dates)

These dates do not change regardless of Boys Option A outcome:

| Date | Event |
|------|-------|
| April 29 – May 5 | Boys Option A spot check run window |
| May 5 (EOD) | Boys Option A decision deadline — if Presten has not run by May 5, default to FAIL for planning purposes |
| May 9 | DSS readiness go/no-go |
| May 14 (noon) | Pre-demo final check — no implementation changes after this point |
| May 16 | DSS demo |

**Critical constraint:** Club Rankings implementation must begin no later than **May 6** (Branch A) or **May 6** (Branch B) to complete before the May 9 go/no-go. Any verdict arriving after May 5 collapses both branches to the same 3-day window — see Section 4.

---

## Section 2: Branch A — Boys Option A PASSES (verdict by May 5)

**Trigger:** ELO files "Boys Option A: PASS" verdict — all three queries passed per `Rankings/Boys Option A Spot Check — Presten Execution Package.md` Decision Table.

**Scope:** Full Club Rankings — Boys + Girls — per `Rankings/Club Rankings — Implementation Specification.md`. Estimated session time: 10–11 hours.

| Date | Owner | Action |
|------|-------|--------|
| Verdict day (≤ May 5) | ELO | File "Boys Option A: PASS" in briefing. Update DSS Readiness Tracker — Club Rankings Boys cell: ✅. Note: Girls-Only Fallback Spec deactivated. |
| Verdict day | SENTINEL | Authorize FORGE for full Club Rankings implementation (Boys + Girls). Reference: `Rankings/Club Rankings — Implementation Specification.md`. |
| May 6–7 (target) | FORGE | Full implementation session: 10–11 hours. Pre-flight Gates A and B both required. |
| May 8 | ELO | Post-implementation validation: run pre-flight confirmation queries from Section 0 of the Implementation Spec, then DSS Data Readiness validation queries (Steps 3A–3C from Validation Plan). File calibration verdict. |
| May 9 | ELO | DSS Final Assessment — Club Rankings cell updated: ✅ Girls + Boys, or ❌ if validation fails. |

**Risk note:** If Boys Option A verdict arrives on May 5, FORGE has 3 days before the May 9 go/no-go. The full 10–11 hour session requires a single uninterrupted block. **This is achievable only if FORGE blocks May 6 in full.** If the May 6 session cannot be a full day, partial implementation is the fallback: complete Girls-only scope first (first 7.5 hours of the spec), then add Boys if time permits. Do not leave a partial Boys implementation that doesn't pass Gate B validation — either complete Boys fully or defer to post-DSS.

---

## Section 3: Branch B — Boys Option A FAILS (verdict by May 5)

**Trigger:** ELO files "Boys Option A: FAIL" verdict — any of the three queries failed per `Rankings/Boys Option A Spot Check — Presten Execution Package.md` Decision Table.

**Scope:** Girls-only Club Rankings — per `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. Estimated session time: 7.5 hours.

| Date | Owner | Action |
|------|-------|--------|
| Verdict day (≤ May 5) | ELO | File "Boys Option A: FAIL" verdict. Update DSS Readiness Tracker — Boys cell: ❌, Girls cell: ⚠️ conditional. Send scope delta to FORGE: "Girls-Only Fallback Spec activated — see `Rankings/Club Rankings — Girls-Only Fallback Spec.md`." |
| Verdict day | SENTINEL | Authorize FORGE for Girls-only implementation scope. No Boys work. Reference: `Rankings/Club Rankings — Girls-Only Fallback Spec.md`, Section 1 (Scope Delta). |
| May 6 (target) | FORGE | Girls-only implementation session: 7.5 hours. Pre-flight Gate A required; Gate B skipped (Boys excluded). |
| May 7 | ELO | Girls-only post-implementation validation: run DSS Data Readiness validation queries Steps 3A–3C (Girls rows only). File calibration verdict. |
| May 9 | ELO | DSS Final Assessment — Club Rankings cell updated: ✅ Girls-only, or ❌ if validation fails. Update demo script: Boys rankings excluded from DSS. |

**Scope delta reminder (from Girls-Only Fallback Spec, Section 1):**
- Remove Gate B query (Boys GA ASPIRE game count check)
- Add `AND r.gender = 'F'` to `ranked_teams` CTE in computation SQL
- Skip V4 validation query (Boys baseline)
- Demo narrative becomes: "Girls club rankings" — not "Boys and Girls"

**If verdict arrives on May 5:** Girls-only implementation must complete by May 8 (3-day window). The 7.5 hour session is tighter than Branch A's 10–11 hours — **more likely to complete within a single 3-day window**. May 5 verdict → FORGE books May 6 → ELO validates May 7 → go/no-go May 9. This is achievable.

---

## Section 4: Late Verdict Protocol

If Presten has not run Boys Option A by **May 4 at end of day**:

**ELO action:** File in May 4 briefing: "Boys Option A spot check window closes May 5. No results received as of May 4. If Presten cannot run before May 5, ELO recommends defaulting to Girls-only scope to protect implementation timeline. SENTINEL authorization requested to pre-activate Branch B on May 5 EOD if no results arrive."

**SENTINEL action:** Review ELO's recommendation. Options:
1. Extend window by 1 day (May 6 verdict) — only if FORGE can still complete implementation by May 8
2. Default to Branch B Girls-only on May 5 EOD without Boys Option A results

**Hard default:** If no Boys Option A results by **May 5 at EOD**, treat as FAIL. Branch B activates automatically. ELO files "Boys Option A: NOT RUN — defaulting to Girls-only per timeline constraints" in the May 5 briefing. FORGE receives Girls-only authorization immediately.

This is an explicit decision, not a gap. SENTINEL and FORGE do not wait indefinitely for Boys results that may not arrive — the demo date is fixed at May 16.

---

## Section 5: Timeline Summary Card

### Boys PASS → Full Club Rankings (Branch A)

```
Option A verdict received (≤ May 5)
        ↓
ELO files PASS → SENTINEL authorizes FORGE (full scope)
        ↓
May 6–7: FORGE full implementation session (10–11 hours)
        ↓
May 8: ELO post-implementation validation
        ↓
May 9: ✅ Club Rankings go/no-go (Boys + Girls)
```

**Risk:** Late verdict (May 5) = 3-day window for 10–11 hour session. Tight. Requires full-day block on May 6.

---

### Boys FAIL (or default) → Girls-Only Club Rankings (Branch B)

```
Option A verdict received or default triggered (≤ May 5 EOD)
        ↓
ELO files FAIL → SENTINEL authorizes FORGE (Girls-only scope)
        ↓
May 6: FORGE Girls-only implementation session (7.5 hours)
        ↓
May 7: ELO Girls-only validation
        ↓
May 9: ✅ Club Rankings go/no-go (Girls only)
```

**Risk:** Late verdict (May 5) = 3-day window for 7.5-hour session. Manageable. 7.5 hours < one full day.

---

## References

- `Rankings/Club Rankings — Implementation Specification.md` — full scope (Branch A, 10–11 hours)
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — fallback scope (Branch B, 7.5 hours)
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — trigger condition and Decision Table
- `Rankings/Boys Calibration Option A — Interpretation Guide.md` — verdict format and interpretation thresholds
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Club Rankings cells this timeline maintains

*ELO — 2026-04-25*
