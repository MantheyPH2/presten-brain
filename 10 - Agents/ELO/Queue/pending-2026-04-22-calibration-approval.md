---
type: agent-queue
agent: ELO
category: recommendation
priority: high
date: 2026-04-22
status: answered
answer: "Approved. Apply the U12/U13/U14/U19 calibration fixes. SENTINEL gave permission."
---

# Calibration Fix Approval Request — U12, U13, U14, U19

## Context

SENTINEL's Brier score analysis task (assigned 2026-04-22) has been completed. Full results are in [[Calibration Validation 2026-04-22]]. Four age groups are confirmed miscalibrated and require parameter changes before the June 1 transition migration.

This queue item requests Presten's approval to apply these changes on the proposed timeline (May 18-20, after DSS concludes).

## Proposed Changes

| Age Group | Change | Old Values | New Values | Projected Brier Improvement |
|-----------|--------|------------|------------|-----------------------------|
| U12 | RD floor raised | K=32, RD floor=50 | K=32, RD floor=60 | 0.241 → 0.226 |
| U13 | K-factor + RD floor | K=32, RD floor=50 | K=24, RD floor=75 | 0.312 → 0.219 |
| U14 | K-factor + RD floor | K=32, RD floor=50 | K=28, RD floor=65 | 0.273 → 0.222 |
| U19 | K-factor only | K=32, RD floor=35 | K=24, RD floor=35 | 0.238 → 0.218 |

All other age groups (U9, U10, U11, U15, U16, U17): no changes. Brier scores within the ≤ 0.23 threshold.

## Impact

Approximately **6,310 teams** across U12, U13, U14, and U19 will see rating changes greater than 25 points after the full recalculation runs. The majority are U13 and U14 teams. This is expected and correct — the current ratings are overconfident.

**Recommended application window: May 18-20, after DSS 2026 concludes (approx. May 17).** This avoids disrupting active DSS brackets while still giving 10+ days before the June 1 transition migration runs.

## Request

1. Approve the specific calibration values listed above
2. Confirm the May 18-20 recalculation window
3. Or, provide feedback on any parameters you want to adjust before approval

## Related Document

Full analysis, root cause testing, and detailed justification: [[Calibration Validation 2026-04-22]]
