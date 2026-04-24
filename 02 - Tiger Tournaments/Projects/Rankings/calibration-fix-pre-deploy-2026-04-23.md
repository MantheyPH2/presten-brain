---
title: Calibration Fix — Pre-Deploy Checklist (U12/U13/U14/U19)
tags: [calibration, brier-score, pre-deploy, glicko, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: implementation-ready
related_task: task-2026-04-23-u13-u14-calibration-fix
---

# Calibration Fix Pre-Deploy Checklist — U12/U13/U14/U19

> [!info] Status
> **Parameter changes:** Confirmed from [[Calibration Validation 2026-04-22]]. ✅  
> **Code change spec:** Written below — ready for implementation.  
> **Timing:** Apply **after DSS (May 17+)** — NOT before. Affects 6,310+ teams.  
> **Held-out validation:** Pending — see [[calibration-fix-validation-results-2026-04-23]].

---

## Section 1: Exact Parameter Changes

These changes are taken directly from Section 3 of [[Calibration Validation 2026-04-22]].

| Age Group | Parameter | Current Value | New Value | Source |
|-----------|-----------|--------------|-----------|--------|
| U12 | `rd_floor` | 50 | **60** | Brier 0.241 → proj. 0.226 |
| U13 | `k_factor` | 32 | **24** | Brier 0.312 → proj. 0.219 |
| U13 | `rd_floor` | 50 | **75** | Combined fix: meets ≤0.23 threshold |
| U14 | `k_factor` | 32 | **28** | Brier 0.273 → proj. 0.222 |
| U14 | `rd_floor` | 50 | **65** | Combined fix: meets ≤0.23 threshold |
| U19 | `k_factor` | 32 | **24** | Brier 0.238 → proj. 0.218 |
| All others | — | — | **NO CHANGE** | U9-U11, U15-U17 are within threshold |

---

## Section 2: Code Changes in compute-rankings.js

The changes touch two locations. Make changes **age-group-conditional** — do not alter parameters for any age group not listed above.

### Change 1: Update the Calibration Parameter Config Object

Find the age-group parameter lookup (likely near the top of compute-rankings.js or in a loaded config object). The current values should look similar to:

```javascript
// CURRENT (before fix)
const AGE_GROUP_PARAMS = {
  U9:  { k_factor: 32, rd_floor: 60, starting_rating: 1200 },
  U10: { k_factor: 32, rd_floor: 60, starting_rating: 1200 },
  U11: { k_factor: 32, rd_floor: 55, starting_rating: 1200 },
  U12: { k_factor: 32, rd_floor: 50, starting_rating: 1200 },
  U13: { k_factor: 32, rd_floor: 50, starting_rating: 1250 },
  U14: { k_factor: 32, rd_floor: 50, starting_rating: 1250 },
  U15: { k_factor: 32, rd_floor: 45, starting_rating: 1300 },
  U16: { k_factor: 32, rd_floor: 40, starting_rating: 1300 },
  U17: { k_factor: 32, rd_floor: 40, starting_rating: 1350 },
  U19: { k_factor: 32, rd_floor: 35, starting_rating: 1350 },
};
```

Update to:

```javascript
// PROPOSED (after fix — changes in **bold** below)
const AGE_GROUP_PARAMS = {
  U9:  { k_factor: 32, rd_floor: 60, starting_rating: 1200 },
  U10: { k_factor: 32, rd_floor: 60, starting_rating: 1200 },
  U11: { k_factor: 32, rd_floor: 55, starting_rating: 1200 },
  U12: { k_factor: 32, rd_floor: 60, starting_rating: 1200 },  // rd_floor: 50 → 60
  U13: { k_factor: 24, rd_floor: 75, starting_rating: 1250 },  // k: 32→24, rd_floor: 50→75
  U14: { k_factor: 28, rd_floor: 65, starting_rating: 1250 },  // k: 32→28, rd_floor: 50→65
  U15: { k_factor: 32, rd_floor: 45, starting_rating: 1300 },
  U16: { k_factor: 32, rd_floor: 40, starting_rating: 1300 },
  U17: { k_factor: 32, rd_floor: 40, starting_rating: 1350 },
  U19: { k_factor: 24, rd_floor: 35, starting_rating: 1350 },  // k: 32→24
};
```

> [!note] If config is loaded from calibration.json rather than hardcoded, edit `config/calibration.json` (or copy `config/calibration-staging-2026-04-23.json` to `config/calibration.json`) instead of editing the JS file.

### Change 2: RD Floor Enforcement

Find the Glicko-2 RD initialization and floor enforcement (Phase 2 of the ranking engine — rating deviation management). It should include a minimum RD clamp:

```javascript
// CURRENT — example pattern (exact code may differ)
function enforceRdFloor(rd, ageGroup) {
  const floor = AGE_GROUP_PARAMS[ageGroup]?.rd_floor ?? 50;
  return Math.max(rd, floor);
}
```

No change to the function logic is needed — it reads from `AGE_GROUP_PARAMS`. The Change 1 update above is sufficient for RD floor changes.

Verify the RD floor is applied in:
- Team initialization (new teams start at `starting_rd` which should be ≥ floor)
- End of each rating period (RD should not drop below floor)

If the floor is hardcoded inline (e.g., `Math.max(rd, 50)`) rather than reading from params, find those inline values for U12, U13, U14 and update them.

### Change 3: K-Factor Application

Find where the K-factor multiplier is applied in Phase 3 (Elo Computation) and Phase 4 (H2H Adjustments):

```javascript
// CURRENT — example pattern
const K = AGE_GROUP_PARAMS[ageGroup]?.k_factor ?? 32;
const ratingChange = K * (actualScore - expectedScore);
```

If K is read from `AGE_GROUP_PARAMS`, Change 1 is sufficient — no additional edits needed here.

If K is hardcoded as `32` anywhere for specific age groups, update those specific values for U13, U14, U19.

---

## Section 3: Pre-Deployment Checklist

Work through in order on deploy day (May 17+):

### Before Touching Any Files

- [ ] Confirm DSS is over and division placements are finalized
- [ ] Check no pipeline run is in progress: `ps aux | grep compute-rankings | grep -v grep`
- [ ] Back up current calibration config: `cp config/calibration.json config/calibration-backup-$(date +%Y%m%d).json`
- [ ] Save ratings baseline snapshot (SQL below)

```sql
-- Baseline snapshot before changes
SELECT age_group, gender, COUNT(*) AS team_count, 
       ROUND(AVG(rating)::numeric, 1) AS avg_rating,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating) AS median_rating,
       MIN(rating) AS min_rating, MAX(rating) AS max_rating
FROM rankings
WHERE age_group IN ('U12', 'U13', 'U14', 'U19')
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

### Apply the Changes

- [ ] Copy staging config to production: `cp config/calibration-staging-2026-04-23.json config/calibration.json`
  - OR: apply code changes to `compute-rankings.js` per Section 2 above
- [ ] Verify diff looks correct: `diff config/calibration.json config/calibration-staging-2026-04-23.json`
  - Expected: no differences (they should now match)

### Run the Recomputation

- [ ] Run full ranking recompute: `node scripts/compute-rankings.js`
  - Estimated runtime: [document from prior runs]
  - All age groups recompute — scoped runs not required since changes only affect 4 age groups

### Post-Deployment Verification

- [ ] Re-run baseline snapshot query — compare before/after
  - U12 median: expected to change slightly (±5–15 pts toward mean)
  - U13 median: may shift (±15–30 pts) — ratings were overconfident in both directions
  - U14 median: may shift (±10–25 pts)
  - U19 median: minor shift (±5–15 pts)
  - **Flag if any median changes by > 50 points** — investigate before proceeding

- [ ] Confirm API returns updated data:
  ```bash
  curl "http://localhost:3000/api/rankings?age_group=U13&gender=F&limit=5"
  ```

- [ ] Request Presten sign-off before marking deployed in the production runbook

---

## Section 4: Rollback Plan

If anything goes wrong:

```bash
# Restore backup
cp config/calibration-backup-YYYYMMDD.json config/calibration.json

# Re-run recompute to restore prior state
node scripts/compute-rankings.js
```

Rollback is safe as long as the backup file exists. Do not delete backup until new calibration has been running 7+ days without issues.

---

## Section 5: Monitoring Week 1 Post-Deploy

| Day | Check | Target |
|-----|-------|--------|
| Day 1 | Run post-apply Brier spot-check on recent test games | U13 < 0.25 (improving direction) |
| Day 3 | Re-run baseline snapshot, compare to Day 1 | Median stable (±5 pts) |
| Day 7 | Full Brier score check on post-deploy games | U13 ≤ 0.219, U14 ≤ 0.222 |
| Day 7 | Confirm no unusual ranking complaints from users | N/A |

Post-apply Brier monitoring SQL is in `Reports/calibration-held-out-validation-2026-04-23.md` Section 4 (the weekly monitor query).

---

## Section 6: Sign-Off

| Step | Status |
|------|--------|
| Brier score analysis reviewed | ✅ Complete (see [[Calibration Validation 2026-04-22]]) |
| Held-out validation run | ☐ Pending (see [[calibration-fix-validation-results-2026-04-23]]) |
| Presten approval received | ☐ Pending (see `Queue/pending-2026-04-22-calibration-approval.md`) |
| Changes implemented in code | ☐ Pending |
| Post-deploy verification complete | ☐ Pending |

**Do NOT deploy until Presten approval is confirmed.**

---

## Related

- [[Calibration Validation 2026-04-22]] — source of truth for parameter values
- [[calibration-fix-validation-results-2026-04-23]] — held-out Brier validation results
- [[Calibration Production Runbook — May 2026]] — full production runbook
- `config/calibration-staging-2026-04-23.json` — staging config with proposed values
- [[Ranking Engine]] — Glicko-2 phases, K-factor and RD implementation
