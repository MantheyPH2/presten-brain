---
title: Source Submission MVP — Session Plan and Schedule
tags: [product, pipeline, mvp, session-plan, dss, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready-to-schedule
task: task-2026-04-26-source-submission-mvp-session-plan
estimated_hours: 4–6
proposed_session_window: "May 1–3 (TBD — Presten to confirm)"
latest_acceptable_date: 2026-05-05
---

# Source Submission MVP — Session Plan and Schedule

> Converts the implementation plan into a scheduled, milestone-tracked session. Goal: MVP ships before DSS (May 16, 2026) so community members can submit sources during the event.

**Implementation plan:** `Product & Planning/Source Submission MVP — Implementation Plan.md` (read before session — 10 min)  
**DSS date:** May 16–18, 2026 (20 days from today)

---

## Section 1: Scheduled Session Window

**Proposed window: May 1–3, 2026 (any single 4–5 hour block)**

Rationale for this window:
- April 28: DB execution session — no capacity for feature work
- April 29: ELO analysis session (G1–G4 gates) — no capacity for feature work
- April 30: Daily pipeline deployment prep and/or first pipeline run monitoring
- **May 1–3: First available window post-deployment**

If May 1–3 is unavailable: **latest acceptable date is May 5**. This leaves 11 days before DSS for testing, QA, and any follow-up fixes. A May 6+ session compresses the buffer below acceptable levels.

**TBD — Presten to confirm the specific date.**

> [!important] Gate condition
> The Source Submission MVP session can run in parallel with daily pipeline monitoring. There is no dependency on the May 1 pipeline launch succeeding — the MVP uses only the `source_submissions` table (independent of game/team data). The session CAN proceed even if the daily pipeline has a YELLOW or RED status, unless Presten's full attention is required for pipeline recovery.

---

## Section 2: Pre-Session Checklist

Complete all items before writing the first line of code.

- [ ] **Reviewed implementation plan** — open `Product & Planning/Source Submission MVP — Implementation Plan.md`, skim Steps 1–5. Estimated read time: 10 minutes.
- [ ] **Database accessible** — `psql -U p -d youth_soccer` connects without error
- [ ] **Server running** — `curl http://localhost:3000` returns a response (not connection refused)
- [ ] **`source_submissions` table does NOT yet exist** — `psql -U p -d youth_soccer -c "\d source_submissions"` returns "Did not find any relation"
- [ ] **`server.js` is editable** — confirm path and write access
- [ ] **No active pipeline run in progress** — check `ps aux | grep crawl` or cron log before starting DB work
- [ ] **Source Submission MVP is independent of SnapSoccer** — SnapSoccer is a separate, deferred work item; it is NOT a prerequisite for this MVP

---

## Section 3: Session Milestones

| Milestone | Description | Estimated Time | Success Signal |
|-----------|-------------|----------------|----------------|
| **M1** | `source_submissions` table created, migration run | 15 min | `\d source_submissions` in psql shows 17 columns including `status DEFAULT 'pending'` |
| **M2** | `source-detector.js` module added | 25 min | Node.js REPL test: `detectSource('https://gotsport.com/games/schedule?id=99999')` returns `{ family: 'gotsport', autoRoutable: true }` |
| **M3** | `POST /api/submit-source` endpoint live | 60 min | `curl` POST with valid GotSport URL returns HTTP 202 with `submission_id` and `source_family` fields |
| **M4** | `GET /api/submissions` admin endpoint live | 20 min | `curl http://localhost:3000/api/submissions` returns JSON array with the test row from M3 |
| **M5** | Submission form renders in browser | 60 min | `http://localhost:3000/submit-source.html` opens, form is visible with URL field, submit button enabled |
| **M6** | End-to-end test complete | 30 min | Form submit with valid GotSport URL → 202 success message displayed → row in `source_submissions` via psql |

**Total: ~3.5–4 hours to M6.** Buffer 30–60 min for debugging.

**Milestone escalation rule:** If any milestone takes more than 2× its estimate (M3 → 120+ min, M5 → 120+ min), pause and flag to FORGE. We decide whether to continue or defer M5/M6 to a second session. A partial MVP (endpoints live, no form) ships in ~2 hours if needed.

---

## Section 4: Post-Session Acceptance Criteria

MVP is **shipped** when ALL of the following are true:

1. `psql -U p -d youth_soccer -c "SELECT COUNT(*) FROM source_submissions WHERE status='pending'"` returns at least 1 (Presten's test submission exists)
2. `POST /api/submit-source` with a GotSport URL returns HTTP 202 and `source_family: "gotsport"`
3. `GET /api/submissions` returns that row with correct `source_type`
4. Web form at `/submit-source.html` shows a success message after Presten submits a test URL
5. All existing endpoints still work — `curl http://localhost:3000/api/h2h-violations` returns normal response (not 500)
6. FORGE files a completion note in the next briefing with the admin review URL

**NOT required for MVP ship:**
- Email confirmation to submitter
- Auto-routing to scrapers
- Admin UI (psql is sufficient for MVP review)
- Authentication on `/api/submissions`
- Community credit system

---

## Section 5: DSS Relevance

**What this MVP enables before DSS:**

Having a source submission form live at DSS (May 16–18) means:
- Tournament directors and coaches at DSS who know of missing game sources can submit them in real time — during demo sessions, at the booth, or after any conversation about coverage gaps
- SENTINEL can present "we have a source submission pipeline" as one of the competitive differentiators vs USA Rank (USA Rank has no community sourcing mechanism)
- Every submission before DSS feeds directly into FORGE's source gap inventory, giving Presten a concrete list of community-prioritized sources to act on post-DSS
- The form URL can be shared in any pre-DSS outreach (newsletter, social, direct club contact) to build the submission backlog before DSS even begins

**What the MVP does NOT need to do before DSS:**
- Process submissions automatically
- Integrate with the crawl pipeline automatically (manual review via psql is acceptable)
- Have a polished admin UI

---

## Section 6: Post-Session Actions (FORGE)

After Presten confirms the session is complete and acceptance criteria are met:

1. Mark `task-2026-04-26-source-submission-mvp-session-plan` as completed
2. Update `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Source Submission status → SHIPPED
3. File a note in next FORGE briefing: "Source Submission MVP shipped. Form available at `[URL]`. Test submission confirmed. Admin review via `psql` or `GET /api/submissions`. [N] submissions received as of filing."
4. Add Source Submission MVP to SENTINEL's DSS readiness section with confirmed ship date

---

## References

- `Product & Planning/Source Submission MVP — Implementation Plan.md` — step-by-step implementation (read before session)
- `Infrastructure/Server and Frontend.md` — server.js structure and how routes are added
- `Infrastructure/Database Schema.md` — `youth_soccer` DB connection details
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — tracks this MVP as a DSS feature

*FORGE — 2026-04-26*
