---
title: NAL (North American League) — Platform Identification Research
tags: [data-sources, research, nal, platform-identification, evo-draw, forge]
created: 2026-04-27
author: FORGE
status: complete — platform unconfirmed; browser check protocol included
task: task-2026-04-27-nal-platform-identification-research
recommendation: HOLD — browser check required before any engineering commitment
---

# NAL (North American League) — Platform Identification Research

**Researched by:** FORGE  
**Date:** 2026-04-27  
**Task:** `task-2026-04-27-nal-platform-identification-research.md`  
**Scope:** Platform identity, data accessibility, volume estimate, and build path for NAL. Research conducted from vault documents and structural inference. Browser confirmation required for definitive platform ID.

> **Note on naming:** Vault documents use "NAL (North American League)" throughout. The task file uses "National Academy League (NAL)." These may refer to different organizations or reflect a name evolution. The North American League in US youth soccer context refers to a US Youth Soccer (USYS)-affiliated development league. This document assumes that identity. Browser confirmation will resolve naming definitively.

---

## Section 1: Platform Identity

**Platform:** **Unconfirmed.** FORGE cannot confirm the NAL platform from vault research alone.

**Primary hypothesis: GotSport** (medium confidence)

Reasoning:
- NAL is affiliated with the US Youth Soccer (USYS) ecosystem
- USYS state associations are overwhelmingly on GotSport — all major USYS state orgs in the Texas, Florida, New York, New Jersey, Washington, and California markets use GotSport
- National league scheduling entities under USYS typically use the same platform as their member state associations
- If NAL is a USYS-administered national league (not a standalone company), it almost certainly has a GotSport org_id
- The vault's Non-GotSport synthesis notes: "search GotSport for 'North American League' or 'NAL Soccer'" as the first browser check step — this implies GotSport is the primary hypothesis

**Confidence:** Medium. FORGE has not confirmed via browser search. There is a non-trivial possibility that:
1. NAL is on a custom or alternative platform (e.g., Blue Star/Demosphere)
2. NAL does not publish public game results at all (internal scheduling only)
3. "NAL" refers to a league that operates under a parent org_id already in the crawl config

**How to confirm (browser check — 10 minutes):**

1. Navigate to `system.gotsport.com` → Admin Login page
2. In the URL or org search field, search for: "North American League" and "NAL Soccer"
3. If a GotSport organization appears with an org_id: **GotSport confirmed** — record org_id
4. If no GotSport result: Navigate to the NAL official website and check URL structure for platform branding (Blue Star, Demosphere, custom)
5. Check whether results/schedule pages are publicly linked from the NAL website

---

## Section 2: Data Accessibility

**If GotSport (primary hypothesis):**
- Same public API access as all other GotSport sources
- `gotsport.com/api/v1/team_ranking_data?org_id={ORG_ID}` — fully public, no auth required
- Zero scraping complexity; existing pipeline handles ingestion natively
- Data quality: same as all GotSport-sourced games (team names, scores, dates, division)

**If non-GotSport:**
- Accessibility unknown — depends on platform
- If Demosphere: HTML-scraping required; no API; moderate complexity
- If custom/static site: likely low volume and inconsistent publishing; research would require additional time
- If no public results publication: BLOCKED — data is inaccessible without partnership/data export

**Rate limiting (if GotSport):** Same GotSport API rate limits as existing pipeline. No additional constraints.

**Auth wall (if GotSport):** None — GotSport org-level data is public.

---

## Section 3: Coverage Estimate

**Games/year estimate: < 1,000 (low confidence)**

Basis for estimate (vault prior):
- The 2026-04-26 Non-GotSport synthesis notes "volume estimated < 1,000 games/year based on league size heuristics (low national footprint vs. established regional leagues)"
- NAL appears to be a smaller national development league — not a full-season regional league like EDP or NPL
- If structured as a national championship/showcase format (similar to Elite 64), event count is low and game volume per season is correspondingly low

**Geographic scope:** National in name, but likely limited in participating clubs. Compared to regional full-season leagues (EDP: ~150+ clubs in Northeast; USL Academy: 90 clubs nationally), NAL's club count is not confirmed from vault research.

**Age groups:** Presumed multi-age-group (U13–U19) based on comparable national development leagues.

**Confidence on volume estimate:** LOW. The < 1,000 figure is a heuristic, not a data-derived count. If NAL turns out to have a significant club count (50+ clubs across multiple age groups), volume could be 2,000–5,000 games/year — still below EDP or USL Academy but above the current estimate.

---

## Section 4: Build Path

### If GotSport (primary hypothesis)

**Complexity: TRIVIAL** — same config-add path as EDP, USL Academy, and all other GotSport sources.

Steps:
1. Confirm org_id via browser lookup at `system.gotsport.com`
2. Open `Infrastructure/GotSport Org-ID Config Edit Reference.md`
3. Add NAL entry to crawl config using League Entity template
4. No engineering work required — pipeline handles natively

**Time required:** 5 minutes (config add) + 30-second verification query.

**Reuse from existing parsers:** 100% — GotSport pipeline requires no modification.

### If non-GotSport

**Complexity: HIGH** — depends entirely on platform. New scraper required regardless of platform. Estimated 20–50+ build hours before the platform is identified.

**Reuse potential:** Low unless the platform turns out to be SincSports (then SnapSoccer parser reuse applies) or Demosphere (no existing parser; medium build).

---

## Section 5: Recommendation

**Recommendation: HOLD — browser check required**

| Scenario | Verdict |
|----------|---------|
| GotSport confirmed via browser | **GO immediately** — trivial config add, no engineering. Upgrade to Tier A in Stage 2 queue. Org_id added within 5 minutes of confirmation. |
| Non-GotSport, volume > 2,000/year | **HOLD** — assess platform, estimate build, re-score. If Demosphere or SincSports, a build sprint may be worthwhile. |
| Non-GotSport, volume < 1,000/year | **NO-GO** — volume does not justify a new scraper build. Deprioritize indefinitely. |
| No public results published | **NO-GO** — inaccessible. Document in Source Gap Inventory as "no public data." |

**If GotSport:** NAL immediately jumps ahead of SnapSoccer and Affinity Soccer in the post-DSS queue — it becomes a 5-minute config add requiring zero engineering. This is the highest-value possible outcome of this research.

**Post-DSS engineering queue impact:**
- Current queue: SnapSoccer (#1) → Affinity Soccer research (#2) → Affinity Soccer build (#3)
- If NAL = GotSport: NAL inserts above SnapSoccer as a zero-effort addition. Complete in the same session as any browser-confirmed org-IDs.
- If NAL ≠ GotSport and volume < 1,000: queue unchanged. NAL moves to indefinite deferral.

---

## Browser Check Protocol

**Time required:** 10–15 minutes  
**Can be combined with:** The browser lookup session already planned for EDP, USL Academy, Elite 64, and 11 other entities

**Steps:**

1. Open `system.gotsport.com` in browser
2. Search for "North American League" and/or "NAL Soccer" in the org lookup field
3. Record result: org_id if found, "not on GotSport" if not found
4. If not on GotSport: search "nal-soccer.com" or "northamericanleague.com" for official website
5. On NAL website: check whether schedule/results links exist and what they load (GotSport embed, Demosphere, custom HTML, or no results published)
6. File result with FORGE per: `Infrastructure/GotSport Browser Session — Results Intake Form.md` (add NAL row)

**FORGE response on confirmation (within 30 minutes):**
- If GotSport: update `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` Section 2 NAL row, add to Stage 2 Config, file in next briefing
- If not GotSport: update Master Reference Section 3 (not on GotSport), update Source Gap Inventory, file in next briefing

---

## Vault Status Summary

| Field | Current Status |
|-------|---------------|
| Platform | **Unconfirmed** — GotSport is primary hypothesis |
| Org_id | Unknown — pending browser confirmation |
| Volume estimate | < 1,000 games/year (low confidence) |
| Build path | 5 minutes if GotSport; unknown/HIGH if not |
| In browser session prep | YES — listed in prep package; combine with existing 11-entity lookup |
| Stage 2 queue position | #10 in priority matrix (Score 3); upgrades to Tier A immediately if GotSport confirmed |

---

## References

- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — NAL section (prior assessment)
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — Section 2 NAL row (pending)
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — NAL included in session
- `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` — NAL placeholder
- `Infrastructure/Source Coverage Priority Matrix — April 2026.md` — Score 3 ranking

*FORGE — 2026-04-27*
