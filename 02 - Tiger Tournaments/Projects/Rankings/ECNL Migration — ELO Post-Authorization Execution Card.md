---
type: elo-execution-card
topic: ecnl-migration-post-authorization
date: 2026-04-29
authorization_deadline: 2026-04-30
status: pre-staged
elo_recommendation: Option 1
tags: [elo, ecnl, migration, execution, authorization]
---

# ECNL Migration — ELO Post-Authorization Execution Card

> **Authorization received from Presten: Option `[ ] 1 — No Re-tag` / `[ ] 2 — Re-tag ecnl_legacy`**
>
> ELO executes the relevant path below within 30 minutes of authorization receipt.
> If authorization arrives with conditions outside Options 1 and 2 as defined, flag to SENTINEL before executing.

---

## Path A — Option 1 Authorized (No Re-tag)

*ELO-recommended path. Total execution time: ~15 minutes.*

**Step 1 — Update Decision Brief (2 min)**
- Open `Rankings/ECNL Migration Option — Decision Brief.md`
- Change frontmatter `status:` from `pending-sentinel-authorization` to `authorized`
- Add at top of Section 1: `> **AUTHORIZED by Presten: [date/time]. Option 1 selected. FORGE proceeding.**`

**Step 2 — Confirm Handoff Checklist reflects Option 1 (5 min)**
- Open `Rankings/ECNL Migration — ELO-FORGE Handoff Checklist.md`
- Confirm Path A: no team re-tagging; FORGE activates event classifier for `ecnl_rl` new events only; CP1 monitors ingestion continuity (not rating continuity)
- No changes needed if checklist already reflects Option 1 — note "Checklist confirmed, no changes" in this document

**Step 3 — Set CP1 Checkpoint Date in Vault (3 min)**
- CP1 date: June 1 + 48 hours = **June 3, 2026**
- Open `Rankings/ECNL Migration — Option Comparison Matrix.md`
- Set CP1 entry: "June 3 — FORGE confirms game ingestion, then ELO runs CP1 analysis"
- Open `Rankings/ECNL CP1 — Checkpoint Results.md`; confirm header states CP1 target date: June 3

**Step 4 — Update May 9 DSS Risk Register (3 min)**
- Open `Rankings/May 9 DSS Gate — Risk Register.md`
- Find Risk R3 (ECNL migration delayed past May 9)
- Update Probability to **Low** and Mitigation to: "OPTION 1 SELECTED — No recompute; FORGE classifier update only; no May 9 window exposure"
- Note DSS Block status as: **NO** (Option 1 migration does not touch May 9 rankings)

**Step 5 — Notify SENTINEL in next briefing (1 min)**
- Include in next ELO briefing: "ECNL Option 1 authorized by Presten on [date]. CP1 set for June 3. DSS risk R3 downgraded to LOW. FORGE may proceed."

**Option 1 complete. Total: ~14 minutes.**

---

## Path B — Option 2 Authorized (Re-tag to ecnl_legacy / ecnl_rl)

*Non-recommended path. Total execution time: ~25 minutes.*

**Step 1 — Update Decision Brief (2 min)**
- Open `Rankings/ECNL Migration Option — Decision Brief.md`
- Change frontmatter `status:` to `authorized-option2`
- Add at top of document: `> **⚠ AUTHORIZED by Presten: [date/time]. Option 2 selected. NOTE: ELO recommendation was Option 1. Option 2 chosen by Presten. FORGE proceeding with re-tag path.**`

**Step 2 — Draft CP1 Checkpoint Plan for Option 2 (10 min)**
- Option 2 CP1 tests schema integrity after FORGE ALTER TABLE + UPDATE. ELO must review rating distribution before and after.
- Open `Rankings/ECNL CP1 — Checkpoint Results.md`
- Add Option 2-specific Section 1 rows:
  - Mass UPDATE row count for historical ECNL games: `> X rows affected` (exact count pending FORGE)
  - Rating continuity: Sample 10 Path A teams — expected drop magnitude 40–80 pts (document as expected, not anomalous)
  - False negatives: Teams that should be re-tagged but were not
- ELO cannot complete CP1 until FORGE ALTER TABLE runs. Note in document: "CP1 HOLD — awaiting FORGE ALTER TABLE execution."

**Step 3 — Activate ECNL Rating Continuity Spec (3 min)**
- Open `Rankings/ECNL Rating Continuity Spec — June 2026.md`
- Mark status as `active` in frontmatter
- Set CP1 and CP2 checkpoint sequence dates:
  - CP1: June 1–3 (post-FORGE ALTER TABLE)
  - CP2: June 7–9 (post-migration rating check)

**Step 4 — Update May 9 DSS Risk Register (3 min)**
- Open `Rankings/May 9 DSS Gate — Risk Register.md`
- Find Risk R3 (ECNL migration delayed past May 9)
- Update to: "OPTION 2 SELECTED — MEDIUM RISK. Full recompute required. FORGE schema change creates CP1 dependency. If FORGE recompute runs in May 9 readiness window, rating anomalies possible."
- Note if May 9 DSS Block now: **CONDITIONAL** (depending on when FORGE recompute runs)

**Step 5 — File SENTINEL Queue Item (5 min)**
- File `10 - Agents/ELO/Queue/pending-2026-04-30-ecnl-option2-selected.md` with:
  - Category: `alert`
  - Priority: `high`
  - Content: "Option 2 selected. FORGE schema change creates CP1 dependency. ELO cannot complete CP1 until FORGE ALTER TABLE runs. Estimated CP1 window: June 1–3. If FORGE recompute scheduled before May 9, ELO needs 48-hour review window before go/no-go gate."

**Step 6 — Notify SENTINEL in next briefing (1 min)**
- Include: "ECNL Option 2 authorized by Presten on [date]. CP1 window: June 1–3. FORGE schema dependency confirmed. May 9 risk R3 elevated to MEDIUM. SENTINEL Queue item filed."

**Option 2 complete. Total: ~24 minutes.**

---

## Decision Record

```
Authorization received: [ ] April 29   [ ] April 30   [ ] Not yet received
Option authorized: [ ] Option 1 — No Re-tag   [ ] Option 2 — Re-tag ecnl_legacy
Authorized by: Presten
Authorization received at: _______________
ELO execution start: _______________
ELO execution complete: _______________
SENTINEL notified: _______________
```

**If no authorization by April 30 EOD:**
- File SENTINEL Queue item: "ECNL option authorization missed April 30 EOD deadline. June 1 migration prep window is now at risk. FORGE cannot begin implementation without Presten authorization. SENTINEL action required."
- Update May 9 DSS Risk Register R3: Probability → **High**

---

*ELO — Pre-staged 2026-04-29. Execute within 30 minutes of Presten authorization.*
