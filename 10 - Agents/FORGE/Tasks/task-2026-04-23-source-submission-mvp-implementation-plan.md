---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
status: completed
completed: 2026-04-23
priority: high
---

# Task: Write Implementation Plan for Source Submission MVP

## Context

Yesterday FORGE designed the Source Submission Framework (design doc at `02 - Tiger Tournaments/Projects/Product & Planning/Source Submission Framework Design.md`). The design is approved. The MVP scope is clear: SQL table + POST endpoint + web form. This task converts that design into a sequenced, hour-by-hour implementation plan that Presten can execute in a single focused session of 4–6 hours.

The goal is zero ambiguity. Presten should be able to open this plan and execute it step-by-step without needing to make any design decisions — all decisions are already locked in the design doc.

Read before starting:
- `02 - Tiger Tournaments/Projects/Product & Planning/Source Submission Framework Design.md` — the design you delivered yesterday. This is your source of truth.
- `02 - Tiger Tournaments/Projects/Infrastructure/Server and Frontend.md` — current Node.js server (port 3000, 5 endpoints, how routes are structured)
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — existing tables, migration conventions
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — Phase 1 action items, "Build a source-submission system" is item #1

## What You Need to Produce

### 1. Pre-Flight Checklist
List everything Presten needs to verify BEFORE starting implementation:
- Confirm PostgreSQL connection string / `youth_soccer` DB access
- Confirm Node.js server is running and `server.js` is editable
- Confirm the existing 5 endpoints still work (GET /api/rankings, etc.)
- Confirm the SQL `CREATE TABLE source_submissions` from the design doc hasn't already been run

### 2. Step-by-Step Implementation Sequence

Break the 4–6 hour session into numbered steps. Each step must include:
- **What to do** (specific action, not "add the endpoint")
- **Exact code to write or run** (copy-paste ready — SQL, Node.js, HTML)
- **How to verify it worked** (a test command or expected output)
- **Time estimate** (minutes)

Required steps (at minimum):
1. Run the `CREATE TABLE source_submissions` DDL against `youth_soccer`
2. Verify table created: `\d source_submissions`
3. Add the `POST /api/submit-source` route to `server.js` — paste the full handler stub from the design doc
4. Test the endpoint: `curl -X POST http://localhost:3000/api/submit-source -H "Content-Type: application/json" -d '{"url":"https://gotsport.com/event/12345","submitter_email":"test@test.com"}'`
5. Add the `GET /api/submissions` admin endpoint (Presten-only, no auth for MVP — document this as a security note)
6. Build the web form (HTML file or inline in existing frontend) — fields: URL, submitter name, email, notes
7. Wire form submit button to `POST /api/submit-source` via fetch
8. Test full flow: submit form → check DB row appears → check API response
9. Add a "Thank you" confirmation message to the form on success

### 3. Code Artifacts (Complete and Ready to Run)

Include the following as complete, copy-paste-ready blocks:

**A. SQL DDL** — final `CREATE TABLE source_submissions` statement (from design doc, no placeholders)

**B. Node.js route handler** — complete `POST /api/submit-source` handler (from design doc, adapted if needed for the actual server.js structure)

**C. `detectSource(url)` function** — the URL-to-source-family classifier from the design doc, ready to paste into server.js

**D. HTML form** — complete form markup, ready to paste into the frontend. Includes: URL input, submitter name, email, optional notes, submit button, and a `<div id="submit-result">` for feedback.

**E. Fetch call** — JavaScript to wire the form submit to the API endpoint, handle success/error, show feedback in `#submit-result`

### 4. Rollback Plan
If something breaks during implementation, what does Presten do?
- How to drop the submissions table if the DDL created it wrong
- How to remove the new routes from server.js without breaking existing routes
- How to verify the 5 existing endpoints still work after changes

### 5. Post-Launch Verification
After the session is complete, Presten should run these checks:
- Submit a test URL via the form — confirm row appears in `source_submissions` with status `pending`
- Query the admin endpoint: `GET /api/submissions` — confirm it returns the test row
- Confirm existing endpoints still return correct data (ranking query test)

### 6. What's Explicitly Out of Scope (MVP)
Document clearly what is NOT in this implementation plan, so Presten doesn't get pulled into scope creep:
- Email notifications to submitters — Phase 2
- Admin UI for reviewing submissions — Phase 2
- Auto-routing to existing scrapers — Phase 2
- Authentication on the admin endpoint — Phase 2 (note it as a known gap)

## What Done Looks Like

Produce an implementation plan at:
`02 - Tiger Tournaments/Projects/Product & Planning/Source Submission MVP — Implementation Plan.md`

The plan must:
1. Be executable in a single 4–6 hour session by Presten alone
2. Contain zero design decisions — all choices already made
3. Have copy-paste-ready code for every code artifact
4. Include verification steps after each major action
5. Have a clear definition of done (what Presten checks at the end to call it shipped)

## Success Criteria

- Plan is written and contains all 6 sections
- Every code artifact is complete — no `// TODO` or placeholder comments
- Presten can start this plan immediately after reading it, with no additional research needed
- The resulting system matches the design doc's MVP spec exactly
- Total implementation time stays within the 4–6 hour estimate from the design doc
