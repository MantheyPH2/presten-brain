---
type: elo-execution-checklist
topic: post-april29-session
date: 2026-04-29
target-completion: "60 minutes post-session"
status: ready
assigned-by: SENTINEL
task: task-2026-04-29-post-session-rapid-execute-checklist
tags: [elo, execution, checklist, april29, session-gate, evo-draw]
---

# Post-April-29-Session — ELO Rapid-Execute Checklist

> **Execute this checklist immediately after Presten completes the April 29 DB session. Target: all steps complete within 60 minutes. Steps are ordered by dependency — do not skip ahead.**

Five tracked items update simultaneously on session execution. This checklist is the single source of truth for post-session ELO actions. Do not improvise — follow in order.

---

## The Checklist

### Step 1 — Confirm GA ASPIRE Fix Applied *(5 min)*

**Action:** Verify the GA ASPIRE event_tier UPDATE was executed and returned an expected row count.

**Confirmation query:**
```sql
SELECT COUNT(*) AS rows_updated
FROM events
WHERE event_tier = 'ga_aspire';
```
Expected: non-zero count matching the April 29 session execution log. If the UPDATE ran correctly, all previously mis-labeled GA ASPIRE events should now carry `event_tier = 'ga_aspire'` instead of `event_tier = 'ga'`.

**If APPLIED:**
- Record confirmation in `Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md` Section 5 — mark exception (1) as **RESOLVED**: "GA ASPIRE fix applied April 29. Post-launch Girls GA ASPIRE ratings reflect correct tier classification."
- Proceed to Step 2.

**If NOT APPLIED:**
- Flag in SENTINEL Queue immediately: "April 29 GA ASPIRE UPDATE did not execute. R1 (GA ASPIRE mis-labeling) remains HIGH in May 9 DSS Risk Register. May 1 monitoring proceeds with pre-fix baseline per Baseline Section 5 exception (1)."
- Do NOT escalate to CONDITIONAL or stop the checklist — continue with Step 2. GA ASPIRE fix is not a blocker for Steps 2–6.

---

### Step 2 — Confirm G0 *(10 min)*

**Action:** Run G0 gate checks per `Rankings/April 29 — Gate Checks Execution Log Template.md`.

G0 is a composite gate. The primary dependency is Girls calibration stability: if the GA ASPIRE fix applied (Step 1 APPLIED), G0 is eligible for GO. Check the full G0 criteria from `Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md` Section 2.

**Record G0 result in briefing:** `GO` or `NO-GO — [reason]`.

**If GO → proceed to Step 3.**

**If NO-GO:**
- Stop and notify SENTINEL via Queue: "G0 = NO-GO. Reason: [state reason]. Event Strength Phase 1 authorization window compressing. Next G0 re-evaluation opportunity: [date of next session]."
- Do NOT fill [TO FILL] cells in Event Strength pre-draft.
- Continue to Step 4 (stddev baseline and Boys Option A are G0-independent).

---

### Step 3 — Fill Event Strength Pre-Draft [TO FILL] Cells *(10 min)*

*Only execute if Step 2 = GO.*

**Action:** Open `Rankings/Event Strength Phase 1 — Authorization Request (Pre-Draft).md`.

Fill the following cells:
- **Section 1, Row 1 (G0 gate):** Actual State → "GO — confirmed [date of April 29 session]"
- **Section 1, Row 2 (G1–G4 girls calibration gates):** Check gate results from session log; enter "PASS" or actual result for each gate
- **Note in document:** "G0 confirmed [date]; G2 gate [result pending / confirmed]; full authorization pending 3-run stability check"

**Do NOT submit to SENTINEL yet.** G2 and the 3-consecutive-run calibration stability check are still pending. Add a note: "G0 confirmed [date]. Submission to SENTINEL pending G2 result and 3-run stability confirmation."

Proceed to Step 4.

---

### Step 4 — Fill Stddev Baseline *(10 min)*

**Action:** Run the psql queries in `Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md` Section 6.

```sql
-- Girls stddev by age group
SELECT age_group, gender, COUNT(*) AS team_count, 
       ROUND(STDDEV(rating)::numeric, 1) AS rating_stddev,
       ROUND(AVG(rating)::numeric, 1) AS avg_rating
FROM rankings
WHERE gender = 'F' AND rank IS NOT NULL
GROUP BY age_group, gender
ORDER BY age_group;

-- Boys stddev by age group
SELECT age_group, gender, COUNT(*) AS team_count,
       ROUND(STDDEV(rating)::numeric, 1) AS rating_stddev,
       ROUND(AVG(rating)::numeric, 1) AS avg_rating
FROM rankings
WHERE gender = 'M' AND rank IS NOT NULL
GROUP BY age_group, gender
ORDER BY age_group;
```

**Fill Section 2** of the baseline document with actual stddev values by age group.

**Remove exception language from Section 5** item (2): "If no April 29 session, ELO runs May 1 morning." Replace with: "April 29 session ran. Stddev baseline values confirmed as filed in Section 2. Exception (2) RESOLVED."

**Mark Section 5 certification** exception (2) as **RESOLVED**.

---

### Step 5 — Initiate Boys Option A *(5 min)*

*G0-independent — execute regardless of Step 2 result.*

**Action:** Confirm B-OA-1/2/3 are ready to run. They are pre-staged in `Rankings/Boys Option A — April 29 Quick-Check Card.md`.

Notify Presten:
> "B-OA-1/2/3 are ready to run. Execution package at `Rankings/Boys Option A — April 29 Quick-Check Card.md`. Estimated 20–30 minutes psql time. This is G0-independent — can run in the same session or any session April 29–May 5."

**If Presten runs queries in the same session:**
- Open `Rankings/Boys Calibration — Option A Verdict Document.md`
- Fill Section 3 (query results table) with actual B-OA-1/2/3 outputs
- Write Section 4 analysis (Boys/Girls delta interpretation, threshold evaluation)
- File within 30 minutes of results receipt
- Filing time target: < 30 minutes from results

**If deferred:** Note in briefing: "B-OA-1/2/3 not yet executed. Execution window: April 29–May 5. Verdict shell standing by."

---

### Step 6 — ECNL CP1 Execution Signal *(5 min)*

**Action:** Confirm ECNL ecnl_verified ALTER TABLE + backfill ran successfully (per April 29 session Step 2b).

**Confirmation query:**
```sql
SELECT COUNT(*) AS ecnl_verified_count, ecnl_verified
FROM teams
WHERE ecnl_verified IS NOT NULL
GROUP BY ecnl_verified;
```
*(Adapt if ecnl_verified is on games table rather than teams table — check schema if needed.)*

**If backfill complete (row count > 0, ecnl_verified column exists):**
- Record confirmation in `Rankings/ECNL CP1 — Checkpoint Results.md` Section 1 — fill Actual column rows 1–4
- Determine CP1 PASS / CONDITIONAL / FAIL per CP1 verdict criteria in that document
- If CP1 PASS: notify FORGE that CP1 data is now available and Option 1 remains viable. FORGE proceeds per ECNL migration scope. Record FORGE notification time in briefing.
- If CP1 CONDITIONAL or FAIL: follow escalation protocol in `Rankings/ECNL Migration — CP1 Fail Escalation Protocol.md`

**If backfill NOT complete:**
- File Queue item to SENTINEL: "ECNL CP1 backfill did not complete in April 29 session. CP1 slipped — will attempt April 30 first action."
- Update `Rankings/ECNL CP1 — Checkpoint Results.md` Section 1: note session date and "CP1 not executed — session dependency not met."

---

### Step 7 — Briefing Update *(15 min)*

**Action:** File the next ELO briefing covering all 6 steps above.

Briefing must include:
- [ ] G0 result (GO / NO-GO with reason)
- [ ] GA ASPIRE fix: APPLIED / NOT APPLIED
- [ ] Event Strength pre-draft [TO FILL] cells: FILLED / DEFERRED (G0 = NO-GO)
- [ ] Stddev baseline: FILLED / DEFERRED (no session) / QUERY EXECUTED — FILLING
- [ ] Boys Option A: INITIATED (queries run) / DEFERRED (window still open)
- [ ] ECNL CP1: PASS / CONDITIONAL / FAIL / SLIPPED TO APRIL 30
- Notify SENTINEL: G0 status, stddev fill status, Boys Option A status

**SENTINEL notification format (include in briefing):**
> "April 29 session complete. G0 = [GO/NO-GO]. GA ASPIRE = [APPLIED/NOT APPLIED]. Stddev baseline = [FILED/DEFERRED]. Boys Option A = [EXECUTED/DEFERRED to [date]]. ECNL CP1 = [PASS/CONDITIONAL/FAIL/SLIPPED]. All items per rapid-execute checklist."

---

## Section 2 — If Session Does Not Open by April 30 EOD

If the April 29 session has not opened by April 30 EOD, ELO must:

1. **Escalate R1 (GA ASPIRE) from HIGH to CRITICAL** in `Rankings/May 9 DSS Gate — Risk Register.md` — update the R1 row: Probability HIGH → HIGH+, add note "Session not opened by April 30 EOD; G0 cannot be confirmed; Event Strength Phase 1 authorization window compressing."

2. **File a Queue item to SENTINEL:**
> "April 29 session still not opened as of April 30 EOD. Event Strength Phase 1 authorization window closes May 7. G0 cannot be confirmed without session. GA ASPIRE fix not applied — Girls post-May-1 pipeline ratings remain mis-classified. ECNL CP1 slipped further. ELO requests SENTINEL flag this to Presten."

3. **Confirm May 1 monitoring proceeds with pre-fix baseline** — per `Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md` Section 5. Monitoring is not blocked. ELO runs stddev baseline query May 1 morning using post-launch data as Day-1 baseline.

---

## Quick Reference

| Step | Action | Time | Dependency | Output |
|------|--------|------|-----------|--------|
| 1 | Confirm GA ASPIRE fix | 5 min | None | Baseline Section 5 exception (1) update |
| 2 | Confirm G0 | 10 min | Step 1 preferred | Gate result; Event Strength path |
| 3 | Fill Event Strength [TO FILL] cells | 10 min | Step 2 = GO | Pre-draft updated |
| 4 | Fill stddev baseline | 10 min | None (G0-independent) | Baseline Section 2 + 5 updated |
| 5 | Initiate Boys Option A | 5 min | None (G0-independent) | Verdict shell ready OR verdict filed |
| 6 | ECNL CP1 signal | 5 min | None | CP1 Checkpoint Results updated |
| 7 | Briefing update | 15 min | Steps 1–6 complete | Briefing filed; SENTINEL notified |

**Total target: 60 minutes.**

This checklist is designed for the session executing without SENTINEL present — ELO executes, then briefs SENTINEL in the next cycle. Do not wait for SENTINEL confirmation between steps.

---

*ELO — 2026-04-29*
