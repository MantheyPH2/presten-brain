---
type: elo-synthesis-brief
topic: boys-calibration
status: conditional-predraft
date: 2026-04-29
verdict-pending: boys-option-a
completion-trigger: "Boys Option A verdict filed"
target-completion: "45 minutes after verdict filing"
task: task-2026-04-29-boys-calibration-synthesis-conditional-predraft
tags: [boys, calibration, dss, synthesis, option-a, club-rankings, evo-draw]
---

# Boys Calibration — Synthesis Brief (Conditional Pre-Draft)

> **This is a conditional pre-draft.** Section 1 is static (applies regardless of verdict). Sections 2A and 2B are divergent paths — fill the relevant one on verdict receipt and delete the other. Sections 3–4 fill within 15 minutes of verdict filing. File within 45 minutes of Boys Option A verdict.
>
> **Filing sequence:** (1) ELO completes Section 1 as part of this task. (2) Boys Option A verdict filed — within 30 min of B-OA-1/2/3 results. (3) ELO fills Section 2A or 2B (15 min). (4) ELO fills Sections 3–4 (10 min). (5) ELO renames file: `Rankings/Boys Calibration — Synthesis Brief.md`. (6) File within 45 minutes of verdict filing.

---

## Section 1 — Boys Calibration State (Static)

*Complete before Boys Option A results arrive. True regardless of verdict.*

### Applied Calibration Settings (as of April 29, 2026)

| Age Group | Setting | Status |
|-----------|---------|--------|
| U13B / U14B | K-factor = 32, RD floor = 50 (current) | **Applied — not yet updated** |
| U13B / U14B (proposed) | K-factor → 24, RD floor → 75 | **DO NOT DEPLOY before May 17** |
| U15B–U17B | K-factor = 32, RD floor = 40–45 | Applied |
| U19B | K-factor change proposed (structural justification only, no Brier validation yet) | Deferred to June 2026 |
| GA ASPIRE Boys | cal = 100 (same as Boys GA) | **Confirmed — no net change from April 28 UPDATE** |
| Boys GA | cal = 100 | Applied |
| MLS NEXT Boys — Homegrown | cal = 160 | Validated (5,800 games); **deploy deferred to May 18–20** |
| MLS NEXT Boys — Academy | cal = 135 | Validated (5,800 games); **deploy deferred to May 18–20** |
| MLS NEXT Boys — Unclassified | cal = 147 | Validated; **deploy deferred to May 18–20** |

### Confirmed Decisions

- **GA ASPIRE Boys (cal = 100):** Decision made April 2026. Boys GA ASPIRE was at cal = 100 (same as Boys GA) before the April 28 UPDATE — the UPDATE reclassifies the event_tier label (`ga` → `ga_aspire`), but Phase 8 of compute-rankings.js uses cal = 100 for both. Net rating impact: **zero**. No April 28 recalibration needed for Boys. Full GA ASPIRE Boys Brier analysis (testing whether cal = 70 would improve accuracy) is deferred to May 17–30.

- **MLS NEXT Boys Tier Split (Deferred May 18–20):** The Homegrown / Academy / Unclassified split is empirically validated from ~5,800 cross-league games. Deployment deferred from April 28 — FORGE event classifier not yet run; adding it to the April 28 session exceeded the ≤ 30-minute session extension threshold. The May 18–20 window is confirmed.

- **U13/U14 K-factor/RD Floor Fix (DO NOT DEPLOY before May 17):** The proposed changes (U13: K→24, RD→75; U14: K→24, RD→75) are structurally justified by the existing blended Brier analysis (U13: 0.312, U14: 0.273 — both above the 0.24 threshold). The Boys-specific Brier pre-check must pass before deployment. Execution window: Boys Brier pre-check May 10–16; deployment decision May 17.

### Known Gaps

- **No gender-separated Boys Brier analysis yet run.** The existing Brier scores (U13: 0.312, U14: 0.273) are blended Boys + Girls. Whether Boys are driving the high score, or Girls (pre-April-28 GA ASPIRE mis-labeling), is unknown until the Boys Brier pre-check runs May 10–16.
- **U15B/U16B not yet assessed** — not planned for DSS. MLS NEXT calibration for U15B–U17B is the strongest empirically (cross-league win-rate validated), but no Brier validation is filed. Full validation deferred to June 2026.
- **U17B–U19B** — K-factor change proposed but not Brier-validated for Boys specifically. Lower DSS exposure. Deferred to June 2026.

### Girls Calibration Reference (State Parity as of April 29 @ 08:58)

- **Girls GA ASPIRE fix:** NOT APPLIED as of this writing. April 29 session not yet opened. If fix applies today, Girls post-session ratings will reflect correct GA ASPIRE tier (cal = 100, reduced from effective cal = 140 via mis-labeling). G0 gate re-evaluates after fix.
- **Girls G1–G4 calibration gates:** On HOLD pending G0 confirmation.
- **Boys vs. Girls calibration parity:** Boys are ahead in some respects (GA ASPIRE decision already made, MLS NEXT validated) but behind in Brier validation. Girls' primary remaining gap (GA ASPIRE) is mechanical — one session away from resolution. Boys' primary remaining gap (Option A, Brier) requires DB access and analysis.

### Timeline Anchors

| Date | Event |
|------|-------|
| April 29–May 5 | Boys Option A queries (B-OA-1/2/3) window — G0-independent |
| April 30 EOD | ECNL option decision due (unrelated to Boys, but same session) |
| May 1 | Pipeline launch — Boys ratings enter active monitoring |
| May 9 | DSS go/no-go gate — Boys Club Rankings included/excluded |
| May 10–16 | Boys Brier pre-check execution window |
| **May 17** | **U13/U14 K-factor/RD floor fix deploy window opens** |
| May 18–20 | MLS NEXT Boys tier split deployment |
| May 30 | Boys GA ASPIRE Brier analysis (cal = 100 vs. 70 decision) |
| June 1 | Age group transition — birth year → school year format |

---

## Section 2A — If Boys Option A APPROVED

*(Fill within 15 min of APPROVE verdict. Then delete Section 2B.)*

**What APPROVE means for boys calibration confidence:**
[TO FILL: Summary of what Option A approval means — e.g., all age group deltas ≤ 75 pts, Boys distribution structurally aligned with Girls, known elite clubs in expected rank positions. Calibration design confirmed empirically for DSS-relevant age groups.]

**Next calibration steps in priority order (first 3):**

1. [TO FILL: e.g., "Boys Brier pre-check May 10–16 — PROCEED per execution package. No blockers. Execution package filed at `Rankings/Boys Brier — Pre-May-17 Execution Package.md`."]
2. [TO FILL: e.g., "MLS NEXT Boys tier split — deploy window May 18–20. FORGE event classifier preflight required by May 17."]
3. [TO FILL: e.g., "Boys GA ASPIRE Brier analysis — window May 17–30. Confirm cal = 100 or revise to 70."]

**Boys Club Rankings go/no-go — updated recommendation:**
[TO FILL: With Option A APPROVED, Boys Club Rankings are [INCLUDED / INCLUDED WITH CAVEAT] in DSS demo. State ELO's recommendation for the May 9 gate.]

**MLS NEXT Boys tier split — does APPROVE change the May 18–20 timeline?**
[TO FILL: Option A APPROVE = Boys distribution is healthy. Does this accelerate or confirm the May 18–20 timeline? Or is the timeline unchanged?]

**SENTINEL recommendation (what ELO is asking SENTINEL to authorize or note):**
[TO FILL: e.g., "ELO requests SENTINEL note Boys Club Rankings as INCLUDED for DSS. Boys Brier pre-check May 10–16 is authorized per filed execution package. May 17 deploy window is confirmed open pending Brier PASS."]

---

## Section 2B — If Boys Option A REJECTED or CONDITIONAL

*(Fill within 15 min of REJECT or CONDITIONAL verdict. Then delete Section 2A.)*

**Which age groups failed and calibration trust implication:**
[TO FILL: State which age groups produced delta > 100 pts (REJECT) or 76–100 pts (CONDITIONAL). State the implication — e.g., "U13B delta of [N] pts indicates Boys ratings in this age group are systematically divergent from Girls. This cannot be explained by known biological factors alone and suggests a data issue (game weighting, merge artifact, or mis-classified events) or a structural Boys underweighting in the current calibration."]

**Girls-only Club Rankings fallback — operational meaning:**
[TO FILL: Per `Rankings/Club Rankings — Girls-Only Fallback Spec.md`, if Boys Option A FAILS, Boys Club Rankings are excluded from DSS. Girls Club Rankings proceed normally. State: FORGE notified same session (see Step 5 of verdict shell), demo script updated to remove Boys rows, DSS demo impact is [minimal / notable — explain].]

**Revised Boys calibration priority queue — what to re-evaluate:**
[TO FILL: After REJECT, ELO must identify the root cause. Priority re-evaluation order: (1) check for merge artifacts in U13B/U14B top-25 — are high-rated teams valid teams or merge ghosts? (2) Check Boys MLS NEXT distribution for compression — is the elite tier underweighted? (3) If merge artifacts: FORGE notification for merge cleanup before next Option A attempt. If MLS NEXT compression: May 18–20 deployment may partially resolve; consider pulling forward.]

**FORGE notification status:**
[TO FILL: Confirm FORGE notified same session per verdict shell protocol. State: "FORGE notified [date/time] that Boys Club Rankings are excluded from DSS. Girls-only fallback spec is active. No Boys ranking rows in May 9 demo."]

**SENTINEL recommendation:**
[TO FILL: e.g., "ELO requests SENTINEL review the REJECT finding. Root cause investigation will file by [date]. Boys Club Rankings excluded from DSS. ELO will propose next Option A attempt timeline once root cause is identified — tentatively May 5–9 if merge artifacts are addressable, or post-DSS if MLS NEXT deployment is the fix."]

---

## Section 3 — ELO Confidence Statement

*(Fill within 10 min of verdict receipt.)*

[TO FILL: One paragraph on ELO's overall confidence in Boys calibration quality given the Option A outcome. State clearly — Presten uses this for the May 9 DSS briefing. Example frameworks below.]

*Pre-draft (APPROVE path):*
Boys calibration confidence is **MEDIUM-HIGH** for DSS. Option A APPROVE confirms that Boys and Girls distributions are within 75 pts across all tested age groups — the structural design is validated at the spot-check level. The Brier pre-check (May 10–16) is the remaining gate for the May 17 K-factor/RD fix. Until the Brier pre-check passes, ELO treats Boys calibration as structurally sound but not predictive-accuracy validated. At DSS on May 9, Boys rankings are safe to display — the risk of showing an embarrassingly wrong team in the top 10 is LOW based on Option A results. The risk of a Boys-specific systematic error remains unknown until Brier runs.

*Pre-draft (REJECT path):*
Boys calibration confidence is **LOW** for DSS. Option A REJECT indicates a material Boys/Girls distribution divergence in at least one DSS-relevant age group. ELO cannot confirm Boys rankings are safe to show without root-cause identification. Girls Club Rankings can proceed; Boys Club Rankings are excluded from the May 9 demo. Confidence will not be restored until (a) merge artifacts are cleaned, or (b) MLS NEXT Boys tier split is deployed and a subsequent Option A spot check passes. Timeline for confidence restoration: [fill based on root cause].

---

## Section 4 — SENTINEL Authorization Items

*(Fill within 10 min of verdict receipt. Maximum 5 items.)*

[TO FILL: Bullet list of items ELO is asking SENTINEL to review or authorize based on boys calibration synthesis. Fill after verdict.]

*Pre-draft (APPROVE path):*
1. **Authorize Boys Club Rankings for DSS demo (May 9)** — ELO recommends inclusion. Decision: SENTINEL note / Presten confirm. Deadline: May 8.
2. **Note Boys Brier pre-check window (May 10–16)** — Execution package filed. No SENTINEL action required; note that failure of Brier pre-check would block May 17 deploy. Deadline: May 17.
3. **Note MLS NEXT Boys tier split (May 18–20)** — Preflight check required by May 17. SENTINEL should confirm FORGE preflight is on the May 17 agenda. Deadline: May 17.

*Pre-draft (REJECT path):*
1. **Authorize Girls-only Club Rankings for DSS demo (May 9)** — Boys excluded per Option A REJECT. Decision: SENTINEL note. Deadline: May 8.
2. **Review Boys Option A REJECT root cause investigation** — ELO filing by [date]. SENTINEL review before next Option A attempt. Deadline: May 9.
3. **Authorize next Boys Option A attempt timeline** — ELO proposes [date range]. SENTINEL approves or redirects. Deadline: [fill based on root cause].

---

## References

- `Rankings/Boys Calibration — Option A Verdict Document.md` — verdict document this brief synthesizes
- `Rankings/Boys Calibration Status Summary — April 2026.md` — baseline state (Section 1 source)
- `Rankings/Boys Brier — Pre-May-17 Execution Package.md` — Brier pre-check (runs May 10–16)
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — activated on REJECT verdict
- `Rankings/Boys Calibration Gap Analysis — April 2026.md` — query thresholds and gap analysis
- `Rankings/MLS NEXT Boys Tier Split — April 28 Decision.md` — MLS NEXT deployment context
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys cell this brief feeds

*ELO — 2026-04-29 (Section 1 complete; Sections 2–4 fill on verdict receipt)*
