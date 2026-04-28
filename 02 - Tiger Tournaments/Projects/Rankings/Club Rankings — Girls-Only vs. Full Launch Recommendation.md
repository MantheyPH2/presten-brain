---
title: Club Rankings — Girls-Only vs. Full Launch Recommendation
type: decision-recommendation
author: ELO
date: 2026-04-27
status: submitted — awaiting SENTINEL/Presten decision
due: 2026-05-01
related: "[[Club Rankings — Girls-Only Fallback Spec]]", "[[Club Rankings — Boys-Conditional Implementation Timeline]]", "[[Boys Option A — Decision Document]]"
tags: [club-rankings, dss, decision, boys, girls, evo-draw]
---

# Club Rankings — Girls-Only vs. Full Launch Recommendation

> **Purpose:** ELO's formal recommendation on whether to launch Club Rankings at DSS (May 16) with Girls-only scope or to wait for a complete Boys + Girls system. SENTINEL and Presten must approve this recommendation before May 1 so the May 6 implementation window is authorized.
>
> **Two paths:**
> - **Path A — Girls-Only at DSS:** Ship Girls Club Rankings at DSS (May 16). Boys follow post-DSS after Boys Option A resolves.
> - **Path B — Wait for Both:** Delay Club Rankings entirely until Boys are ready (late May / early June).

---

## Section 1: Decision Criteria

### Demo Impact

Girls Club Rankings is a compelling standalone DSS asset. The top 5 reference clubs (Eclipse Select, PDA, FC Dallas, PA Classics, NJ Wildcats) are nationally recognized programs that DSS audience members — primarily Girls program directors and Girls travel coaches — will recognize and trust. The narrative is not "here are partial rankings"; it is "here is the definitive national Girls club ranking — who the strongest programs are, from U13 through U17."

Boys absence is not a visible hole for this audience. A Girls program director at DSS will not ask "where are the Boys?" — she will ask whether the rankings match her perception of the Girls landscape.

**Assessment:** Girls-only is a complete, credible demo story for the DSS audience. It is not a fallback; it is appropriate scope for this audience.

### Technical Readiness

Girls Club Rankings is de-risked:
- 5/5 Girls age groups confirmed calibration-validated (post-April-28 GA ASPIRE fix)
- Girls-Only Fallback Spec is written with explicit scope delta, session estimate (8.5 hours), and reference clubs
- Boys-Conditional Implementation Timeline Branch B is executable immediately on Boys FAIL verdict

Boys Club Rankings is conditional:
- Option A spot check has not run yet (window: April 29 – May 5)
- Without Option A results, Boys cannot be authorized for DSS
- If Option A results arrive May 5 (last possible day), FORGE has 3 days (May 6–8) to complete a 10–11 hour full implementation — tight but executable only with a full uninterrupted May 6 block
- If Option A results arrive May 4, there is more margin; if May 3 or earlier, full Boys + Girls is achievable with lower risk

**Does Girls-only scope create debt for Boys integration later?** No. The `club_rankings` table schema is gender-agnostic. Boys are added by removing the `AND r.gender = 'F'` filter from the `ranked_teams` CTE and re-running Gate B. This is one SQL line change. Girls-only implementation does not lock in any architecture that makes Boys harder to add.

**Assessment:** Girls scope is fully de-risked. Boys scope is conditional on a result that does not exist yet. Girls-only launch creates zero technical debt against Boys integration.

### Boys Timeline Risk

**Probability of Boys Option A resolving in time for DSS inclusion:** Medium.

The Option A window closes May 5. The probability of Boys passing by May 5 (enabling Branch A):
- Best case (results arrive April 29): Branch A PASS → full implementation May 6–7 → validated May 8 → go/no-go May 9. Clean.
- Expected case (results arrive May 3–4): Branch A window tightens but still executable.
- Worst case (results arrive May 5): Branch A requires full-day FORGE block on May 6 with zero slip margin before May 9 go/no-go.
- Boys FAIL case: Branch B activates immediately. Girls-only 8.5-hour session completes May 6 → validated May 7 → go/no-go May 9. This is the lower-risk path.

The risk is not that Boys Option A fails — it's that Boys Option A results arrive late OR pass with a Conditional verdict requiring SENTINEL disposition, which adds a decision round-trip that may consume the May 5–6 window.

**Assessment:** Boys timeline risk is real. Girls-only path eliminates timeline uncertainty entirely.

### Calibration Dependency

Girls Club Rankings has no unresolved calibration dependency after the April 28 GA ASPIRE fix. All 5 Girls age groups: Calibration validated ✅, post-fix stability monitoring underway ✅.

Boys Club Rankings requires Option A results to confirm Boys calibration is not structurally off. Option A spot check (3 queries: Boys/Girls divergence, Boys GA game volume, MLS NEXT Boys distribution) is the calibration gate for Boys Club Rankings.

**Assessment:** Girls are ready. Boys have an open calibration gate.

---

## Section 2: Recommendation

**ELO recommends Path A — Girls-Only Club Rankings at DSS.**

**Reasoning:**

Girls Club Rankings is calibration-confirmed, implementation-ready, and audience-appropriate. The DSS demo can open with "Here are club rankings for Girls programs — a national view of which programs are strongest from U13 through U17" without qualification. Eclipse Select, PDA, and PA Classics in the top 15 will anchor the rankings for the NJ/PA audience.

Boys Club Rankings cannot be authorized without Option A results, and those results cannot arrive before April 29 at the earliest. Even in the best case, Boys Club Rankings adds implementation risk and timeline pressure (10–11 hours vs. 8.5 hours, with a full-day FORGE block required if results arrive May 5). Waiting for Boys to authorize Girls would hold back a de-risked, audience-appropriate feature.

Path B (wait for both) sacrifices a confirmed DSS asset on the chance that Boys also makes it in. If Boys fails or results arrive late, DSS has no Club Rankings at all. This is the worse outcome.

**Conditional addition of Boys:** This recommendation does not foreclose Boys Club Rankings at DSS. If Boys Option A returns a PASS verdict by May 3, and FORGE can confirm a full-day May 6 session, Boys can be added to the implementation scope without displacing Girls. The Boys-Conditional Implementation Timeline Branch A is still the target. ELO is recommending Girls as the guaranteed baseline — not Girls as the ceiling.

**Specific condition that changes the recommendation:** If Boys Option A results arrive by April 30 with a clean PASS on all three queries AND FORGE confirms May 6 availability, ELO will update this recommendation to Path A + Conditional Boys (Branch A). This condition must be confirmed by May 1 to avoid holding up FORGE's scope definition.

---

## Section 3: Implementation Trigger

**Girls-only implementation begins immediately after SENTINEL/Presten approves this recommendation.**

1. ELO confirms Girls-only scope delta to FORGE: `Rankings/Club Rankings — Girls-Only Fallback Spec.md` Section 1 is the implementation spec (one-line SQL change to `ranked_teams` CTE, Gate B skipped, V4 removed).
2. FORGE books implementation session for May 6 (8.5 hours: pre-flight, Phase 1–4, validation).
3. ELO runs post-implementation validation queries May 7 (V1 Girls count ≥ 50 clubs, V2 top 10 recognizable, V3 PA Classics ≤ rank 30).
4. Boys Club Rankings: ELO continues monitoring Option A results through May 5. If PASS arrives by May 3 and FORGE has May 6–7 capacity, ELO requests SENTINEL authorization to extend to Branch A (full Boys + Girls). If not, Boys Club Rankings is filed for post-DSS sprint (targeting June 2–15 per the Sprint Plan).

**If Boys PASS arrives after this recommendation is approved:** SENTINEL makes the call on whether to extend scope to Branch A or hold Girls-only. ELO does not unilaterally expand FORGE's scope — this requires explicit authorization from SENTINEL after reviewing Branch A timeline feasibility.

---

## Section 4: SENTINEL/Presten Decision Required

ELO is requesting one of the following:

**Option 1 — Approve Path A (Girls-only at DSS):** FORGE proceeds with 8.5-hour Girls-only session May 6. Boys Club Rankings targeted post-DSS sprint (June). No further decision needed until Boys Option A results arrive.

**Option 2 — Approve Path A + Conditional Boys:** Approve Girls-only as the baseline AND authorize ELO to escalate to Branch A (full Boys + Girls) if Boys Option A PASSES by May 3. ELO monitors and notifies SENTINEL within 24 hours of Option A results arriving. If Branch A is triggered, FORGE session extends to 10–11 hours on May 6.

**Option 3 — Approve Path B (wait for both):** Hold Club Rankings for DSS entirely. Target a dual-gender Club Rankings launch post-DSS (late May or early June). ELO does not recommend this path, but defers to Presten if the DSS audience is broader than Girls-focused.

**Deadline for this decision: May 1.** FORGE needs scope clarity before May 6 implementation window.

---

## References

- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — Girls-only scope delta, session estimate, reference clubs, readiness assessment
- `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md` — Branch A and Branch B timelines with explicit dates
- `Rankings/Boys Option A — Decision Document.md` — Boys calibration gate (unfilled until April 29–May 5 results arrive)
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — the three queries that determine Boys Option A verdict
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Club Rankings cell to update after this decision

*ELO — 2026-04-27*
