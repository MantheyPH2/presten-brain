---
title: Daily Pipeline Log Format Spec
tags: [infrastructure, pipeline, logging, run-daily, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: spec-ready — implement when writing run-daily.sh
task: task-2026-04-24-daily-pipeline-log-spec
companion: "Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md (Section 1.3)"
---

# Daily Pipeline Log Format Spec

**Purpose:** Define the log format for `run-daily.sh` so that:
1. The ECNL Migration Pre-Flight Checklist Section 1.3 ("3 consecutive successful runs") can be verified programmatically.
2. Pipeline failures are unambiguous and diagnosable from the log alone.
3. Presten and FORGE can confirm pipeline health without reading raw output.

**When to use:** Reference this document when writing `run-daily.sh` (due May 1, 2026 per the Option A implementation plan in `Product & Planning/Daily Pipeline Cadence Scope.md`).

---

## Section 1: Log File Location

### Path Convention

```
logs/daily-pipeline-YYYY-MM-DD.log
```

- Relative to the project root (same convention as `Logs/gotsport-sync/` for crawl logs).
- One file per calendar day. The filename date is the date the run **started**, not ended.
- Example: a run starting at 2026-05-01 02:00 AM → `logs/daily-pipeline-2026-05-01.log`

### Create the Directory

If `logs/` does not exist when `run-daily.sh` is first deployed, create it:

```bash
mkdir -p logs
```

Add this as Step 0 in `run-daily.sh` setup instructions. The directory must exist before the first run, or the log redirect will fail silently.

### Rotation Policy

- Retain daily log files for at least **30 days**.
- Files older than 30 days may be archived or deleted.
- Do NOT rotate mid-run — one file per day is sufficient.

### Note on Alternate Path

The Option A spec in `Product & Planning/Daily Pipeline Cadence Scope.md` uses `/var/log/evo-draw/pipeline-$(date +%Y%m%d).log`. Either path is acceptable, but **use the project-relative `logs/` path** for consistency with the existing `Logs/gotsport-sync/` convention. If the server uses `/var/log/evo-draw/`, update the verification command below to match the actual path.

---

## Section 2: Required Log Lines

### Successful Run Format

Every successful run of `run-daily.sh` must emit the following lines **in this order**, writing to the daily log file:

```
[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE STARTED
[YYYY-MM-DD HH:MM:SS] STEP 1: Crawl started (team IDs: N)
[YYYY-MM-DD HH:MM:SS] STEP 1: Crawl completed (games fetched: N, errors: N)
[YYYY-MM-DD HH:MM:SS] STEP 2: Rankings compute started
[YYYY-MM-DD HH:MM:SS] STEP 2: Rankings compute completed (teams ranked: N)
[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE COMPLETED SUCCESSFULLY
```

### Failed Run Format

Every failed run must emit:

```
[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE STARTED
...
[YYYY-MM-DD HH:MM:SS] ERROR: [component] failed — [error message]
[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE FAILED (exit code: 1)
```

### Key Rule: Last-Line Success Marker

**The last line of a successful run is always `DAILY PIPELINE COMPLETED SUCCESSFULLY`.**

The pre-flight verification command (Section 3) works by checking for this exact string in the log. If the run was interrupted mid-stream, the last line will be something other than the success marker — and the check will correctly report failure.

### Bash Implementation in `run-daily.sh`

```bash
#!/bin/bash
set -e

LOG_FILE="logs/daily-pipeline-$(date +%Y-%m-%d).log"
exec >> "$LOG_FILE" 2>&1

log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

log "DAILY PIPELINE STARTED"

# --- Step 1: Active team crawl ---
ACTIVE_IDS=$(bash get-active-team-ids.sh)
ID_COUNT=$(echo "$ACTIVE_IDS" | tr ',' '\n' | wc -l | tr -d ' ')
log "STEP 1: Crawl started (team IDs: $ID_COUNT)"
GSAPI_TEAM_IDS="$ACTIVE_IDS" node crawl-gotsport-api-parallel.js
log "STEP 1: Crawl completed (games fetched: see SYNC_RUN_SUMMARY above, errors: see DLQ)"

# --- Step 2: Full pipeline (Steps 1–9) ---
log "STEP 2: Rankings compute started"
bash run-pipeline.sh
log "STEP 2: Rankings compute completed (teams ranked: see compute-rankings output above)"

log "DAILY PIPELINE COMPLETED SUCCESSFULLY"
```

> [!note] Exit Code
> The `set -e` at the top ensures any step failure causes an immediate non-zero exit, preventing the `DAILY PIPELINE COMPLETED SUCCESSFULLY` line from being written. If a step fails, the `DAILY PIPELINE FAILED` line must be written manually (or via a `trap ERR` handler). See below.

### Failure Handler (trap)

Add a failure trap to ensure the `DAILY PIPELINE FAILED` line is always written on error:

```bash
# Add after the log() function definition:
trap 'log "DAILY PIPELINE FAILED (exit code: $?)"' ERR
```

With this trap, if any step exits with a non-zero code, the failed marker is logged automatically before the script exits.

---

## Section 3: Verification Command

### Check Last 3 Daily Pipeline Logs for Success

Use this command to confirm 3 consecutive successful runs (as required by the ECNL Migration Pre-Flight Checklist Section 1.3):

```bash
#!/bin/bash
# Verify last 3 daily pipeline runs completed successfully
# Run from the project root

PASS=0
FAIL=0

for i in 1 2 3; do
  # macOS date syntax
  DATE=$(date -v -${i}d '+%Y-%m-%d' 2>/dev/null)
  # Linux fallback
  [ -z "$DATE" ] && DATE=$(date -d "${i} days ago" '+%Y-%m-%d' 2>/dev/null)

  LOGFILE="logs/daily-pipeline-${DATE}.log"

  if [ -f "$LOGFILE" ] && grep -q "DAILY PIPELINE COMPLETED SUCCESSFULLY" "$LOGFILE"; then
    echo "${DATE}: PASS"
    PASS=$((PASS + 1))
  else
    echo "${DATE}: FAIL (log missing or pipeline did not complete)"
    FAIL=$((FAIL + 1))
  fi
done

echo ""
if [ "$FAIL" -eq 0 ]; then
  echo "RESULT: All 3 checks PASSED — pre-flight Section 1.3 satisfied"
else
  echo "RESULT: $FAIL of 3 checks FAILED — do NOT proceed with ECNL migration"
fi
```

**Save this as `scripts/check-pipeline-health.sh`** or run inline. The last `echo` line is the go/no-go output for the pre-flight check.

### Quick One-Liner (for manual checks)

```bash
for i in 1 2 3; do
  D=$(date -v-${i}d '+%Y-%m-%d' 2>/dev/null || date -d "${i} days ago" '+%Y-%m-%d');
  grep -q "DAILY PIPELINE COMPLETED SUCCESSFULLY" "logs/daily-pipeline-${D}.log" 2>/dev/null \
    && echo "${D}: PASS" || echo "${D}: FAIL"
done
```

> [!note] OS Compatibility
> - **macOS:** uses `date -v -${i}d`
> - **Linux:** uses `date -d "${i} days ago"`
> - The commands above include both forms with fallback. If running on Linux only, the macOS form can be removed.

---

## Section 4: Minimum Requirements for `run-daily.sh` Author

When Presten writes `run-daily.sh`, the script **must**:

1. **Create or confirm the log directory** before writing to it (`mkdir -p logs`).
2. **Redirect all stdout to the daily log file** — not just print to terminal.
   ```bash
   LOG_FILE="logs/daily-pipeline-$(date +%Y-%m-%d).log"
   exec >> "$LOG_FILE" 2>&1
   ```
3. **Write `DAILY PIPELINE STARTED` as the first logged line** (after the `exec` redirect).
4. **Write either** `DAILY PIPELINE COMPLETED SUCCESSFULLY` **or** `DAILY PIPELINE FAILED (exit code: N)` **as the final line** — the `trap ERR` handler handles failures automatically if implemented.
5. **Exit with code 0** on success. **Exit with code 1** on any step failure (`set -e` handles this).

### Connecting Node.js Steps to the Log

Since `crawl-gotsport-api-parallel.js` and `run-pipeline.sh` are separate processes, their stdout must also go to the log file. The `exec >> "$LOG_FILE" 2>&1` redirect at the top of the script handles this — all child process output flows to the log automatically.

---

## Section 5: Step Definitions (for the Log Lines)

The log lines reference "STEP 1" and "STEP 2." These map to:

| Log Step | Scripts Run | Source of Truth |
|---|---|---|
| STEP 1: Crawl | `get-active-team-ids.sh` + `crawl-gotsport-api-parallel.js` | `Daily Pipeline Cadence Scope.md` Section 6 |
| STEP 2: Rankings compute | `run-pipeline.sh` (Steps 1–9: normalization, dedup, compute-rankings) | `Data Pipeline.md` — Steps 1–9 |

If `run-daily.sh` runs intermediate steps explicitly, add log lines for each — e.g., `STEP 1a: Normalization complete`, `STEP 1b: Dedup complete`. The critical requirement is that `STEP 2: Rankings compute completed` appears before `DAILY PIPELINE COMPLETED SUCCESSFULLY`.

---

## References

- `Product & Planning/Daily Pipeline Cadence Scope.md` — Option A spec; `run-daily.sh` template (Section 6)
- `Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md` — Section 1.3 uses the verification command from this doc
- `Infrastructure/Server and Frontend.md` — project server conventions
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage 3 sequencing constraint (references this log format for confirmation)
- `Logs/gotsport-sync/` — existing crawl log convention (this doc follows the same project-relative path pattern)

---

*FORGE — 2026-04-24*
