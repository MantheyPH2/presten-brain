---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md"
---

# Task: Club Rankings — Boys-Conditional Implementation Timeline

## Context

Two Club Rankings documents exist:
- `Rankings/Club Rankings — Implementation Specification.md` — full Boys + Girls scope, 10–11 hour session estimate
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — Girls-only scope if Boys Option A fails, 8.5 hour session estimate

What neither document contains: a timeline. Both specs define *what* to build. Neither specifies *when* to build it, given a May 16 DSS deadline and a Boys Option A decision window that closes May 5.

This creates a planning gap: if Boys Option A results come in on May 3 (PASS), how many days does FORGE have for implementation? If results come in on May 2 (FAIL), when does ELO hand the fallback spec to FORGE? When must the implementation session start to complete before the May 9 go/no-go?

This task: write the conditional timeline. One document, two branches, all dates explicit.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md`

---

### Section 1: Timeline Anchors (Fixed Dates)

These dates do not change regardless of Boys Option A outcome:

| Date | Event |
|------|-------|
| April 29 – May 5 | Boys Option A spot check window |
| May 5 | Boys Option A decision deadline (if Presten has not run by May 5, treat as FAIL for planning purposes) |
| May 9 | DSS readiness go/no-go |
| May 14 | Pre-demo final check (no implementation changes after noon) |
| May 16 | DSS demo |

---

### Section 2: Branch A — Boys Option A PASSES (by May 5)

Trigger: ELO files "Boys Option A: PASS" verdict no later than May 5.

Timeline for full Club Rankings (Boys + Girls):

| Date | Owner | Action |
|------|-------|--------|
| Day 0 (verdict date) | ELO | Update DSS Readiness Tracker — Boys cell: ✅. File briefing noting Girls-Only Fallback Spec is deactivated. |
| Day 0 | SENTINEL | Authorize FORGE to begin full Club Rankings implementation session. |
| Day 1–2 (target: May 6–7) | FORGE | Implementation session: full Club Rankings per spec. Duration: 10–11 hours. |
| Day 3 (target: May 8) | ELO | Post-implementation validation: run readiness queries. File calibration verdict. |
| May 9 | ELO | DSS Final Assessment Spec — Club Rankings cell updated. |

**Risk note:** If Boys Option A verdict does not arrive until May 5, FORGE has 3 days to implement before May 9. The full 10–11 hour session requires a single uninterrupted block. State whether this is achievable given typical session length, or whether partial implementation is the fallback if verdict arrives late.

---

### Section 3: Branch B — Boys Option A FAILS (by May 5)

Trigger: ELO files "Boys Option A: FAIL" verdict. Girls-Only Fallback Spec is activated immediately.

Timeline for Girls-only Club Rankings:

| Date | Owner | Action |
|------|-------|--------|
| Day 0 (verdict date) | ELO | File FAIL verdict. Update DSS Readiness Tracker — Boys cell: ❌, Girls cell: ⚠️ conditional. Notify FORGE: "Girls-Only Fallback Spec activated — see `Rankings/Club Rankings — Girls-Only Fallback Spec.md`." |
| Day 0 | SENTINEL | Authorize FORGE for Girls-only implementation scope. No Boys work. |
| Day 1 (target: May 6) | FORGE | Girls-only implementation session. Duration: 8.5 hours (per fallback spec). |
| Day 2 (target: May 7) | ELO | Girls-only post-implementation validation. |
| May 9 | ELO | DSS Final Assessment — Club Rankings cell: Girls-only ✅ or ❌. |

**If verdict arrives late (May 5):** Girls-only implementation must complete by May 8 (3 days). The 8.5 hour session is tighter on this timeline. State whether a May 5 verdict still allows Girls-only completion before May 9.

---

### Section 4: Late Verdict Protocol

If Presten has not run Boys Option A by May 4 and no verdict is filed:

- ELO proactively files in the May 4 briefing: "Boys Option A spot check window closes May 5. No results received. If Presten cannot run before May 5, ELO recommends defaulting to Girls-only scope to protect implementation timeline."
- SENTINEL reviews and decides whether to extend the window or proceed to Girls-only.
- Default: treat as FAIL if no results by May 5 close of business. Girls-only scope activates.

This is an explicit decision, not a gap. ELO and FORGE proceed on Girls-only rather than waiting indefinitely for Boys results that may not arrive.

---

### Section 5: Timeline Summary Card

Write a one-page summary (this section IS the summary) with two columns side-by-side (or two clear blocks):

**Boys PASS → Full Club Rankings**
- May 5 verdict → May 6–7 FORGE session → May 8 ELO validation → May 9 ✅
- Risk: Late verdict (May 5) = 3 days, tight

**Boys FAIL → Girls-Only Club Rankings**
- May 5 verdict → May 6 FORGE session → May 7 ELO validation → May 9 ✅
- Risk: Late verdict (May 5) = 3 days, manageable (8.5h < 11h)

---

## Quality Standard

- All dates are explicit. No "within a few days" or "shortly after."
- Both branches must be complete enough to hand to FORGE on verdict day with no additional planning required.
- The late verdict protocol must define "no results by May 5" as a hard default to Girls-only — not an open question.

## Why This Matters

SENTINEL's job on May 5 is to read ELO's verdict and immediately authorize FORGE. Without this timeline, that authorization is followed by a planning conversation about when FORGE should start and whether the deadline is achievable. With this timeline, SENTINEL's authorization is: "Branch A activated. FORGE starts May 6 per the conditional timeline." No ad-hoc scheduling under a one-week deadline.

## References

- `Rankings/Club Rankings — Implementation Specification.md` — full scope (Branch A)
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — fallback scope (Branch B)
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — trigger condition
- `Rankings/Boys Option A — Results Interpretation Guide.md` — verdict format
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Club Rankings cell
