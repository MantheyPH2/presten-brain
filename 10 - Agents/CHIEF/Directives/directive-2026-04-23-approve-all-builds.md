---
type: directive
from: Presten
to: CHIEF
date: 2026-04-23
priority: urgent
---

# DIRECTIVE: All Builds Approved — Execute Immediately

Presten has approved ALL pending builds. No more design. Start coding.

## Approved for FORGE (push to his Tasks/):

1. **Pipeline structural fixes** — Apply the 4 code changes (exponential backoff, dead letter queue, empty payload detection, upsert logging) to the actual codebase at ~/Desktop/Projects/evo-draw/. Do it now.

2. **Daily pipeline (Option A)** — Implement the active team filter, create run-daily.sh, set up cron. Ship before May 1.

3. **Source submission MVP** — Create the database table, build the POST endpoint, add the web form. Ship by May 8.

4. **SnapSoccer scraper** — Build it. SincSports-based. Ship by May 12.

## Approved for ELO (push to his Tasks/):

5. **Rank bands** — Implement the SQL view, API endpoint, React component. Ship by May 2.

6. **GA ASPIRE calibration fix** — Change calibration from 140 → ~100 for ASPIRE tier. Apply in the May 18-20 batch.

7. **Event strength ratings** — Implement for DSS demo. Start by April 28 latest.

8. **Pipeline structural fixes from FORGE** need ELO's calibration fixes applied in the same batch window (May 18-20).

## For SENTINEL:

Create implementation tasks for FORGE and ELO based on the above. Each task should reference the design doc and give step-by-step implementation instructions. These are no longer design tasks — they are BUILD tasks. The agents should be writing actual code to ~/Desktop/Projects/evo-draw/.

## Timeline

DSS is May 16. Everything except club rankings ships before then.
