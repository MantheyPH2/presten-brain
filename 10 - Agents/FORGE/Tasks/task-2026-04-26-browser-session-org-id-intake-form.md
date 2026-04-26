---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Browser Session — Results Intake Form.md"
---

# Task: Browser Session Org-ID Results Intake Form

## Context

The browser session is imminent — it could run today, April 27, or April 28. Three existing documents cover the session:

1. **Browser Lookup Session Prep Package** — tells Presten WHAT to look up (11 entities)
2. **GotSport Org-ID Master Reference** — WHERE results land (Section 2, pending-to-confirmed transitions)
3. **GotSport Org-ID Config Edit Reference** — HOW to add confirmed org_ids to the crawl config

What does not exist: a structured form for Presten to fill IN during the session itself. Currently, results come back as narrative text in a briefing note or message. FORGE then parses the narrative to extract org_ids and determine which documents to update. This introduces transcription risk (did FORGE capture all 11 entities? Was "not found" filed correctly or left blank?) and requires FORGE to be available for immediate interpretation.

An intake form eliminates this risk. Presten fills one table during the browser session. FORGE reads one table and updates four documents. The form is a disposable intermediary — its value is capturing the raw session output in a format FORGE can consume without ambiguity.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Browser Session — Results Intake Form.md`

---

### Usage Instructions (top of file)

```
PRESTEN: Open this file before starting the browser session. For each entity:
1. Go to GotSport, search the entity name
2. If found: copy the org_id from the URL or organization page
3. If not found: write "NOT FOUND"
4. Note any ambiguity (multiple results, name variation, etc.) in the Notes column
5. File this document when done — FORGE will update all dependent documents within the same session
```

---

### Results Table

Pull all 11 entities from `Infrastructure/GotSport Browser Lookup Session Prep Package.md` and pre-populate the Entity column. Include the Type column from the prep package.

| # | Entity Name | Type | Found? (Y/N) | Org-ID (if found) | Notes / Ambiguity |
|---|-------------|------|-------------|-------------------|-------------------|
| 1 | [from prep package] | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |
| 6 | | | | | |
| 7 | | | | | |
| 8 | | | | | |
| 9 | | | | | |
| 10 | | | | | |
| 11 | | | | | |

---

### Post-Session Summary (Presten fills)

```
Total entities confirmed: ___/11
Total org-IDs captured: ___
Total not-found: ___
Session duration: ___
Any entities requiring follow-up (ambiguous results): ___
```

---

### FORGE Processing Protocol (FORGE fills after receiving intake)

When this form is filed, FORGE executes in this order:

1. **Update GotSport Org-ID Master Reference:**
   - Move confirmed rows from `pending-browser-lookup` → `confirmed-[org_id]` in Section 2
   - Move confirmed rows to Section 1 (Active Org-IDs)
   - Move not-found rows to Section 3 (Deferred / Not-on-GotSport)
   - Update Section 4 Status Summary counts

2. **Update Stage 2 Authorization Trigger Document:**
   - Section 3: Update browser session status from "pending" to "complete [date]"
   - Count confirmed entities vs. total expected

3. **Update GotSport Org-ID Config Edit Reference:**
   - For each confirmed Tier A source (EDP, USL Academy, Elite 64): note org_id ready to add

4. **Notify SENTINEL:**
   - File one line in the next briefing: "Browser session complete. [N] confirmed, [N] not found. Tier A sources confirmed: [list]. Config update ready for Presten authorization."

---

## Quality Standard

- All 11 entities from the prep package must appear as pre-populated rows — Presten should not have to type entity names
- The "Notes / Ambiguity" column must accept free-text; do not use checkboxes here because GotSport search results vary
- The FORGE Processing Protocol must reference the 4 specific documents to update, in order
- File must be ready before the browser session starts — this task is due April 27 precisely because the session window is today–April 28

## Why This Matters

The browser session is the unlock for 5,000–13,000 additional games/year from three Tier A sources (EDP, USL Academy, Elite 64) at zero engineering cost. The delay risk is not technical — it is information capture. If Presten runs the session and files results as a paragraph of notes, FORGE may miss an org_id or misattribute a "not found" result. An intake form takes 5 minutes for FORGE to write and zero extra time for Presten to fill (it replaces the narrative they were going to write anyway). This is the highest ROI task on FORGE's queue today.

## References

- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — source of 11 entities
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — primary update target
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — Section 3 update
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config update guidance
- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — Tier A sources context
