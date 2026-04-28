---
title: SincSports Browser Check Protocol
tags: [data-sources, sincsports, snapsoccer, browser-check, evo-draw, forge]
created: 2026-04-27
author: FORGE
status: ready-for-execution
---

# SincSports Browser Check Protocol

> Presten runs this check in under 5 minutes. Result is a binary ACCESSIBLE / BLOCKED. Filing the result unlocks the post-DSS SnapSoccer build authorization.

**Why this check matters:** The SincSports Go/No-Go Decision Package (2026-04-27) issued a post-DSS GO for SnapSoccer contingent on one unresolved gate: `soccer.sincsports.com` public schedule pages must be accessible without authentication or Cloudflare blocking. The scraper design assumes public HTML access. If the subdomain is protected, the build cannot proceed without a different access strategy.

---

## Step 1 — Navigate to the Schedule Subdomain

Open a browser and navigate to:

```
http://soccer.sincsports.com/schedule.aspx
```

**What success looks like:** A page loads — even a blank schedule or an error about a missing tournament ID is fine. You are checking that the page responds at all.

**What failure looks like:** A Cloudflare challenge page ("Checking your browser…"), a login wall, or a 403/404 error.

---

## Step 2 — Spot-Check a Known Event

Navigate to a specific event schedule using this URL format:

```
http://soccer.sincsports.com/schedule.aspx?tid=STXSCC25
```

This is a known SincSports tournament TID sourced from `snapsoccer.com/events/`. If this exact TID no longer has data, try:

```
http://soccer.sincsports.com/schedule.aspx?tid=STXSCC26
```

**What success looks like:** An HTML table with game results, team names, dates, and scores. Any schedule table is a pass — it does not need to be populated with recent games.

**What failure looks like:** Cloudflare challenge, login prompt, "Access Denied," or completely blank page with no HTML table structure.

---

## Step 3 — Check for Auth Wall on a SnapSoccer Event Page

Navigate to:

```
https://snapsoccer.com/events/
```

Pick any listed event and click its "Results" or "Schedule" link. Confirm the link resolves to `soccer.sincsports.com/schedule.aspx?tid=XXXX`. Check that the schedule page loads without requiring an account.

**What success looks like:** Schedule page loads from the snapsoccer.com event link without an account login.

**What failure looks like:** Redirect to a login page after clicking the results link.

---

## Pass / Fail Criteria

| Result | Criteria |
|--------|----------|
| **ACCESSIBLE** | Step 1 page responds (any HTML) AND Step 2 returns a schedule table (game rows visible) AND Step 3 links resolve without auth |
| **BLOCKED** | Any step returns Cloudflare challenge, 403, login wall, or completely blank page with no HTML content |

---

## Result Filing Format

After completing the check, file the result by replying with or noting:

```
SincSports browser check result: [ACCESSIBLE / BLOCKED]
Step 1 (subdomain loads): [YES / NO]
Step 2 (schedule table visible): [YES / NO / TID returned no data but page loaded]
Step 3 (snapsoccer.com event link resolves without auth): [YES / NO]
Date/time: [YYYY-MM-DD HH:MM]
Notes: [any observations]
```

**If ACCESSIBLE:** FORGE will close the SincSports accessibility gate and confirm the SnapSoccer post-DSS GO conditions as met (pending May 1 pipeline health confirmation and SENTINEL authorization).

**If BLOCKED:** FORGE will log as NO-GO indefinitely and update the SincSports Go/No-Go Decision Package. SnapSoccer is removed from the post-DSS engineering queue.

---

## References

- `Data Sources/SincSports-GoNoGo-Decision-Package-2026-04-27.md` — decision this check gates
- `Infrastructure/SnapSoccer — Test Extraction Plan.md` — next step after ACCESSIBLE confirmed

*FORGE — 2026-04-27*
