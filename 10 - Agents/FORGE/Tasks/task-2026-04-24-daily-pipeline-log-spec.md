---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Daily Pipeline Log Format Spec.md"
---

# Task: Daily Pipeline Log Format Specification

## Context

FORGE's ECNL Migration Pre-Flight Checklist (Section 1.3) states: "Confirm by: review the pipeline log for at least 3 consecutive successful daily runs before June 1." Two problems:

1. `run-daily.sh` does not exist yet (due May 1, Presten writes it).
2. What constitutes a "successful run" in the log is undefined.

The pre-flight checklist cannot be executed unless the log format is defined before the pipeline is written. If Presten writes `run-daily.sh` without a log format spec, the log may not include the information needed to confirm success — and the June 1 pre-flight check becomes ambiguous.

This task: write the log format spec so that when Presten writes `run-daily.sh`, they know exactly what to log and where.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Infrastructure/Daily Pipeline Log Format Spec.md`

---

### Section 1: Log File Location

Define:
- Expected log file path: `[project root]/logs/daily-pipeline-YYYY-MM-DD.log` (or adapt to match the existing project log conventions from `Infrastructure/Server and Frontend.md`)
- Rotation policy: one log file per day, retained for at least 30 days
- If a `logs/` directory does not exist yet: document the `mkdir -p logs` step in the `run-daily.sh` setup instructions

---

### Section 2: Required Log Lines

Every successful run of `run-daily.sh` must emit the following lines in this order:

```
[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE STARTED
[YYYY-MM-DD HH:MM:SS] STEP 1: Crawl started (team IDs: N)
[YYYY-MM-DD HH:MM:SS] STEP 1: Crawl completed (games fetched: N, errors: N)
[YYYY-MM-DD HH:MM:SS] STEP 2: Rankings compute started
[YYYY-MM-DD HH:MM:SS] STEP 2: Rankings compute completed (teams ranked: N)
[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE COMPLETED SUCCESSFULLY
```

Every failed run must emit:

```
[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE STARTED
...
[YYYY-MM-DD HH:MM:SS] ERROR: [component] failed — [error message]
[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE FAILED (exit code: 1)
```

Key requirement: the last line of a successful run is always `DAILY PIPELINE COMPLETED SUCCESSFULLY`. The pre-flight check script can confirm success by checking whether this line is present in the last 3 daily log files.

---

### Section 3: Verification Command

Write the exact shell command Presten can use to confirm 3 consecutive successful runs:

```bash
# Check last 3 daily pipeline log files for success
for i in 1 2 3; do
  date=$(date -v-${i}d '+%Y-%m-%d' 2>/dev/null || date -d "${i} days ago" '+%Y-%m-%d')
  logfile="logs/daily-pipeline-${date}.log"
  if [ -f "$logfile" ] && grep -q "DAILY PIPELINE COMPLETED SUCCESSFULLY" "$logfile"; then
    echo "${date}: PASS"
  else
    echo "${date}: FAIL (log missing or pipeline did not complete)"
  fi
done
```

Note: adapt the date command syntax for the OS where the pipeline runs (macOS vs Linux). Document which one applies.

This command should be added verbatim to Section 1.3 of the ECNL Migration Pre-Flight Checklist (`Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md`) — update that document as well.

---

### Section 4: Minimum Log Requirements for `run-daily.sh` Author

When Presten writes `run-daily.sh`, the script must:
1. Redirect all stdout to the daily log file (not just print to terminal)
2. Write the `DAILY PIPELINE STARTED` line at the top
3. Write either `DAILY PIPELINE COMPLETED SUCCESSFULLY` or `DAILY PIPELINE FAILED (exit code: 1)` as the final line
4. Exit with code 0 on success, code 1 on any step failure

If the pipeline scripts (crawl, rankings compute) are Node.js processes, recommend using `>> logs/daily-pipeline-$(date +%Y-%m-%d).log 2>&1` redirect.

---

## Definition of Done

- Log file location defined with path and rotation policy
- Required log lines documented (success and failure formats)
- Last-line success marker defined (`DAILY PIPELINE COMPLETED SUCCESSFULLY`)
- Verification command written and tested for syntax correctness (macOS + Linux variants)
- Pre-flight checklist Section 1.3 updated to reference the verification command
- Summary in briefing: "Pipeline log format spec written; pre-flight checklist Section 1.3 updated."

## Why This Matters

If `run-daily.sh` is written without a log spec, the June 1 pre-flight check item "3 consecutive successful runs" cannot be verified programmatically. Without machine-readable success/failure markers, Presten must manually read log files under time pressure before the migration. One spec document written now saves 30 minutes of uncertainty on June 1.

## References

- `02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md` — Section 1.3 references this log; update that section with the verification command
- `02 - Tiger Tournaments/Projects/Infrastructure/Server and Frontend.md` — existing project conventions (confirm log directory location)
- `02 - Tiger Tournaments/Projects/Product & Planning/Daily Pipeline Cadence Scope.md` — pipeline step definitions (what STEP 1, STEP 2 etc. correspond to)
