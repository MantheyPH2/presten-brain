---
type: agent-queue
agent: ELO
category: update
priority: low
date: 2026-04-29
status: pending
answer: ""
tags: [boys, ga-aspire, calibration, league-hierarchy, sentinel-gate]
---

# Boys GA ASPIRE Calibration — Status Update

## Summary

The Boys GA ASPIRE calibration question has been **resolved**. This queue item documents the resolution and flags the remaining open item for June 2026.

## Resolution (April 24, 2026)

**Decision:** Boys GA ASPIRE calibration = **100** (Option A — same as Boys GA).

**Basis:**
- Girls GA ASPIRE miscalibration was at 140 (tagged as `ga`); correct value is 100 — a 40-point error.
- For Boys, the equivalent issue does NOT produce a calibration error because Boys GA is already at cal=100. When Boys GA ASPIRE events are reclassified from `ga` to `ga_aspire`, the calibration lookup returns 100 either way. **Net rating impact on Boys: zero.**
- The April 28 UPDATE (`event_name ILIKE '%ASPIRE%'`) is gender-agnostic and safe to run without Boys-specific modification.

**Cross-league win rate data:**
Insufficient Boys GA ASPIRE cross-league data to empirically derive a differentiated Boys GA ASPIRE calibration value. The only viable derivation would be structural: if Boys GA → Boys GA ASPIRE mirrors the Girls GA (140) → Girls GA ASPIRE (100) ratio, Boys GA ASPIRE would be ~70. However, this is unvalidated and the downside risk (overcalibrating Boys GA ASPIRE teams by ~30 pts) is lower than the Girls case (40 pts).

## League Hierarchy Status

| Circuit | Gender | Cal Value | Status |
|---------|--------|-----------|--------|
| GA ASPIRE | Girls | 100 | Applied (post-April-28 fix) |
| GA ASPIRE | Boys | 100 | Applied (no change from Boys GA; assumed correct) |

Boys GA ASPIRE = 100 is entered in `Calibration Values — League Hierarchy Reconciliation.md` as of April 26. League Hierarchy wikilink reflects this entry.

## Open Item for June 2026

**Boys GA ASPIRE Option B analysis** — whether ~70 is more accurate than 100 for Boys GA ASPIRE. Blocked until:
- Boys Brier analysis (post-DSS, May 17+) provides baseline Boys calibration accuracy
- Sufficient Boys GA ASPIRE game volume available for cross-league analysis

ELO recommends raising this in the Post-DSS Calibration Sprint (June 2026) if Boys Brier scores reveal calibration drift in Boys GA game clusters.

## Source Documents

- `Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md` — full decision analysis
- `Rankings/Calibration Values — League Hierarchy Reconciliation.md` — current calibration table
- `task-2026-04-29-boys-ga-aspire-calibration.md` — originating task
