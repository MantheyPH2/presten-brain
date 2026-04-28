---
title: Post-DSS Sprint — Week 1 Execution Calendar
type: execution-calendar
author: ELO
date: 2026-04-27
status: ready — activate May 17
related: "[[Post-DSS Calibration Sprint Plan]]", "[[Post-DSS Calibration Roadmap — June 2026]]", "[[Post-DSS-Calibration-Priority-Queue-2026-04-27]]"
tags: [sprint, post-dss, calibration, execution, may-2026, evo-draw]
---

# Post-DSS Sprint — Week 1 Execution Calendar

> **Purpose:** Day-by-day execution plan for May 17–24 (Sprint 1 + start of Sprint 2). Converts the Post-DSS Calibration Sprint Plan into specific sessions with owners, hours, dependencies, and decision points. SENTINEL uses this to brief Presten for the May 17 sprint start; no design session required — this is an execution document.
>
> **Written:** 2026-04-27. **Activation:** May 17 (day after DSS). **Sprint Plan reference:** `Rankings/Post-DSS Calibration Sprint Plan.md`.

---

## Section 1: Pre-Sprint Approval Requirements

These items must be approved or confirmed before May 17. ELO flags each item's current status.

| # | Item | Blocks | Authorization Document | Status |
|---|------|--------|------------------------|--------|
| 1 | `compute-rankings.js` U13/U14 code change (K-factor and RD floor params applied for U12, U13, U14, U19) | U13/U14 calibration fix deployment (Day 1, May 17) | `task-2026-04-23-u13-u14-calibration-fix.md` — parameters specified in Section 1 | ⚠️ **NOT YET APPLIED** — Presten must apply the code change before May 17 session. This is the May 17 entry condition per Sprint Plan. No deployment without it. ELO flags this to SENTINEL as a blocker. |
| 2 | Boys Option A verdict (PASS / FAIL / CONDITIONAL) | Boys calibration execution (Day 2, May 18, if PASS) | `Rankings/Boys Option A — Decision Document.md` — ELO fills after results arrive | ⏳ **PENDING** — Window April 29 – May 5. If no verdict by May 5 EOD, ELO defaults to FAIL (per Boys-Conditional Timeline). Not a blocker for Day 1 U13/U14 work. |
| 3 | FORGE event classifier pre-flight for MLS NEXT Boys (≥ 90% classification quality) | MLS NEXT Boys tier split deployment (Day 6, May 22) | FORGE delivers pre-flight results; Presten reviews classification report | ⏳ **PENDING** — FORGE must run classifier before May 17 for May 22 deployment to be on schedule. If FORGE delivers May 18–20, May 22 deployment slips to May 23 (contingency slot available). |
| 4 | Presten confirmation of June 1 `age_group_advance()` mechanism (automatic vs. manual step) | Sprint 3 June 1 sequencing | `Rankings/Post-DSS Calibration Roadmap — June 2026.md` Section 5 | ⏳ **PENDING** — ELO requests confirmation by May 28 per Roadmap. Not a Week 1 blocker, but logging here so SENTINEL can prompt Presten before May 28. |

**Blocking items for May 17 start:**
- Item 1 is the only hard blocker. If `compute-rankings.js` code change is not applied by May 16, the Day 1 deployment session cannot run. ELO requests SENTINEL confirm Presten has applied the code change before May 17 morning.

---

## Section 2: Day-by-Day Calendar (May 17–24)

---

### May 17 (Day 1) — U13/U14 Calibration Fix Deployment

**Task:** Deploy U13/U14 calibration parameter changes and run full recompute.

**Actions:**
1. Take pre-deploy snapshot baseline (Presten runs snapshot query — 15 min). See `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` for snapshot format.
2. Apply K-factor and RD floor code changes to `compute-rankings.js` (already done pre-sprint per pre-condition #1 — this step is running the deployment, not writing the code).
3. Run `compute-rankings.js` full recompute (Presten executes; estimated 30–45 min depending on compute time).
4. Quick spot-check on recompute output: confirm no pipeline errors; check U13 top-10 team list does not show implausible names or ratings. File output in briefing.

**Hours:** 15 min snapshot + 45 min recompute + 30 min spot-check = ~90 min Presten session.

**Dependency:** `compute-rankings.js` code change applied by May 16 (pre-sprint condition #1). Pre-deploy snapshot query ready (same format as April 29 snapshot package).

**Presten session required:** YES — ~90 min. Presten runs snapshot, runs recompute, files output.

**ELO action:** ELO monitors output in the same session (autonomous review — 30 min). Files Day 1 briefing note: "U13/U14 fix deployed. Pre-deploy snapshot taken. Recompute complete. Quick spot-check [pass/fail with specific observation]."

---

### May 18 (Day 2) — Boys Option A Execution (Conditional) + FORGE Pre-Flight Review

**Task A — Boys Option A Execution (if verdict PASS filed April 29–May 5):**

ELO prepares Boys calibration production UPDATE SQL (per `Rankings/Boys Option A — Post-Pass Implementation Timeline.md`). Presten applies the change and runs Boys-targeted recompute.

**Actions:**
1. ELO files Boys calibration UPDATE SQL (autonomous — 30–45 min, done before May 18 session)
2. Presten reviews SQL, authorizes with one-line confirmation in session
3. Presten applies UPDATE and runs targeted Boys recompute

**Hours (Boys execution):** 30 min ELO SQL review + 60 min Presten execution = ~60 min Presten session.

**If Boys Option A verdict is FAIL or not yet filed:** Skip Boys execution. Day 2 is ELO-light. Presten has no session obligation on Day 2.

**Dependency:** Boys Option A PASS verdict filed (Section 1 Item 2). Without a PASS verdict, Day 2 Boys work does not run.

**Task B — FORGE MLS NEXT Pre-Flight Review:**

If FORGE has delivered event classifier results by May 18, ELO reviews the classification quality report (~20 min). If ≥ 90% quality confirmed, ELO files pre-flight clearance in briefing. If FORGE has not yet delivered, ELO notes in briefing and checks again May 19.

**Presten session required:** YES (conditional on Boys PASS) — ~60 min for Boys execution. If no Boys execution, Day 2 is ELO-autonomous only (no Presten session needed).

---

### May 19 (Day 3) — U13/U14 Validation Window (ELO Autonomous)

**Task:** ELO runs U13/U14 post-recompute validation (first validation pass, 48 hours post-deployment).

**Actions:**
1. Check U13 rating distribution: top-10 teams plausible? Median in expected range? No age group suppressed below Gold/Silver floor?
2. Check rating shift magnitude: average U13 shift < 50 pts? Average U14 shift < 50 pts? Flag any age group showing > 50 pt average.
3. Cross-gender check: Girls U13/U14 ratings shifted in same direction as Boys (parameters are gender-neutral — both should show similar shifts)?
4. File validation progress note in briefing. Do NOT declare clearance yet — need 3-day stability window (full clearance May 21).

**Hours:** ~45 min ELO autonomous.

**Dependency:** Recompute complete May 17.

**Presten session required:** NO.

---

### May 20 (Day 4) — Boys Option A Validation (Conditional) + Continued U13/U14 Window

**Task A — Boys Option A Validation (if execution ran May 18):**

ELO runs validation checks on Boys calibration execution results.

**Actions:**
1. Confirm Boys/Girls rating delta ≤ 75 pts in all age groups (re-run Q1 logic from Option A spot check)
2. Confirm Boys MLS NEXT teams still in 1,400–1,700 range (anchor check)
3. Confirm Girls ratings unchanged (cross-contamination check: 3 Girls Gold-band teams before/after Boys recompute)

**Hours:** ~30 min ELO autonomous.

**Task B — U13/U14 Stability Observation (Day 2 of 3):**

No action required; ELO notes stability observation in briefing. Looking for rating oscillation signals.

**Dependency (Boys):** Boys execution complete May 18.

**Presten session required:** NO.

---

### May 21 (Day 5) — U13/U14 Calibration Stability Clearance

**Task:** ELO files U13/U14 calibration stability clearance (3-day stability window complete: May 17, 18, 19).

**Actions:**
1. Review 3-day observation: no rating oscillation, no age group suppressed below expected band, shifts within ≤ 50 pt average
2. File clearance note in briefing: "U13/U14 calibration stability confirmed. [Date deployed: May 17]. Criteria met: [list specific checks passed]. Sprint 2 MLS NEXT deployment cleared from U13/U14 dependency."
3. If any check failed during May 19–21: file "U13/U14 stability flag — [specific issue]." Hold MLS NEXT deployment until resolved. Alert SENTINEL.

**Hours:** ~15 min ELO filing.

**Dependency:** 3-day observation window (May 17–21).

**Presten session required:** NO (unless ELO flags instability — in that case, SENTINEL and Presten must review before May 22 MLS NEXT session).

---

### May 22 (Day 6) — MLS NEXT Boys Tier Split Deployment

**Task:** Deploy MLS NEXT Boys Academy/Homegrown tier split (if FORGE pre-flight cleared).

**Entry condition:** (1) FORGE classifier ≥ 90% quality confirmed AND (2) U13/U14 stability clearance filed May 21.

**Actions:**
1. ELO reviews FORGE pre-flight results one final time (10 min). Confirms ≥ 90% quality. Files authorization in briefing.
2. Presten applies MLS NEXT Boys tier split UPDATE SQL (provided by ELO from `Rankings/MLS NEXT Tier Split — Calibration Application Spec.md`)
3. Presten runs targeted Boys recompute
4. ELO spot-checks post-UPDATE Boys MLS NEXT distribution immediately: Boys Academy average should drop ~20–40 pts; Homegrown should hold. Check that Girls ratings are unchanged.

**Hours:** 10 min ELO pre-flight review + 80 min Presten UPDATE + recompute + 20 min ELO spot-check = ~80–90 min Presten session.

**Dependency:** FORGE pre-flight PASS + U13/U14 stability clearance. If FORGE pre-flight not yet confirmed, Day 6 moves to Day 7 (contingency slot). Do not combine with any other recompute within 48 hours of May 17 Boys recompute (per Sprint Plan 48-hour rule — Boys execution on May 18 means MLS NEXT cannot run before May 20; May 22 is safe).

**Presten session required:** YES — ~80 min for UPDATE + recompute.

---

### May 23 (Day 7) — CONTINGENCY

**No planned tasks.**

This day is reserved for:
- MLS NEXT Boys deployment slip (if FORGE pre-flight not confirmed by May 22)
- Investigation of unexpected rating shifts from U13/U14 fix or Boys Option A execution
- Re-run of any monitoring query showing Yellow/Red
- Presten availability slip on any prior day

**ELO action on contingency day:** File one-line briefing note: "Day 7 contingency. [Active / Used for: specific issue]." If contingency is not needed, file: "Day 7 unused — all Sprint 1 items completed on schedule."

**Presten session required:** NO unless contingency is activated.

---

### May 24 (Day 8) — MLS NEXT Boys Tier Split Validation

**Task:** ELO validates MLS NEXT Boys tier split results (2 days post-deployment).

**Actions:**
1. Boys MLS NEXT Academy average rating: compare pre/post — expect 20–40 pt drop
2. Boys MLS NEXT Homegrown average rating: compare pre/post — expect stable (within 10 pts)
3. Girls ratings unchanged (same cross-contamination check as May 20)
4. No cross-age-group Brier regression signal (top-10 distributions in U13–U17 look plausible)

File: "MLS NEXT tier split validation complete. Academy shift: [N] pts. Homegrown shift: [N] pts. Girls: unchanged. Sprint 2 cleared."

**Hours:** ~30 min ELO autonomous.

**Dependency:** MLS NEXT recompute complete May 22.

**Presten session required:** NO.

---

## Section 3: Contingency Slots

**May 23 (Day 7)** is the designated contingency day (see above). One additional contingency window:

**May 24 PM:** If May 24 validation reveals a MLS NEXT distribution anomaly (Academy teams not dropping, or Girls ratings affected), ELO files an instability flag and holds Sprint 2's Boys GA ASPIRE Brier analysis. Presten is notified for a decision session before May 25.

The contingency structure covers:
- **FORGE pre-flight delay:** MLS NEXT deployment moves May 22 → May 23. Validation moves May 24 → May 25. Week 1 completes one day late but still ahead of May 28 Sprint 2 completion target.
- **Boys Option A Conditional verdict:** ELO holds Boys execution until SENTINEL disposes the conditional. Day 2 session is skipped; Boys validation (Day 4) is also skipped. Sprint 2 Boys work is rescheduled to May 22–24 if disposition arrives by May 21.
- **U13/U14 instability flag:** ELO holds MLS NEXT deployment (Day 6) until resolved. Contingency day (May 23) used for investigation. If not resolved by May 23, ELO flags to SENTINEL and MLS NEXT slips to post-May-24.

---

## Section 4: Week 1 Success Definition

Week 1 (May 17–24) is successful if all of the following are true by end of May 24:

1. **U13/U14 calibration fix deployed and validated.** Post-recompute U13 and U14 distributions show no age group below expected band (Gold ≥ 1,500, Silver ≥ 1,350). Average rating shifts ≤ 50 pts. No pipeline errors. ELO stability clearance filed May 21.

2. **Boys Option A calibration applied** (if PASS verdict filed) **with Boys/Girls delta ≤ 75 pts in all age groups.** Boys MLS NEXT anchor stable (no Boys age group median below 1,300). Girls ratings confirmed unchanged post-Boys recompute.

3. **FORGE MLS NEXT pre-flight cleared at ≥ 90% classification quality.** MLS NEXT Boys tier split UPDATE applied. Post-UPDATE Boys Academy average drops 20–40 pts. Homegrown holds within 10 pts of expected range. Girls unchanged.

4. **No active calibration instability flags.** ELO briefings from May 17–24 show GREEN for U13/U14, Boys (if applied), and MLS NEXT. No "hold pending investigation" flags in any briefing.

5. **Sprint 2 entry conditions met.** Boys GA ASPIRE Brier analysis can begin May 25. Event Strength Phase 2 authorization request prepared (pending Phase 1 validation, which also clears this week). Pre-June-1 calibration state is on track.

---

## References

- `Rankings/Post-DSS Calibration Sprint Plan.md` — source sprint plan this calendar operationalizes
- `Rankings/Post-DSS Calibration Roadmap — June 2026.md` — full roadmap context and sequencing constraints
- `Rankings/Post-DSS-Calibration-Priority-Queue-2026-04-27.md` — priority ordering (do not re-sequence without reason)
- `task-2026-04-23-u13-u14-calibration-fix.md` — U13/U14 code change parameters (pre-sprint condition #1)
- `Rankings/Boys Option A — Post-Pass Implementation Timeline.md` — Boys execution detail for Day 2
- `Rankings/MLS NEXT Tier Split — Calibration Application Spec.md` — MLS NEXT UPDATE SQL and FORGE pre-flight spec
- `Rankings/Boys Brier Analysis Scope Spec — May 2026.md` — Sprint 2 Boys GA ASPIRE Brier (begins May 25)

*ELO — 2026-04-27*
