---
title: Parsing Rules
aliases: [Age Parsing, Gender Parsing, Round Detection, Season Rules]
tags: [pipeline, parsing, normalization, age-group, gender, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Parsing Rules

These rules govern how the [[Data Pipeline]] (Steps 2-5) extracts structured data from raw text fields. Used by `backfill-age-gender.js`, `final-backfill.js`, `backfill-all-games.js`, and `fix-api-gender.js`.

---

## Age Group Parsing Priority

Checked in this order — first match wins:

| Priority | Pattern         | Example Input    | Output |
| -------- | --------------- | ---------------- | ------ |
| 1        | Standard        | "U10", "U-10"    | U10    |
| 2        | Reversed        | "10U"            | U10    |
| 3        | Compact         | "13UB", "11UG"   | U13, U11 |
| 4        | Spanish         | "Sub 17"         | U17    |
| 5        | Under           | "Under 14"       | U14    |
| 6        | Multi-age       | "U13/U14"        | U13 (take lower) |
| 7        | Birth year      | "B2011" (in 2026)| U15    |

> [!tip] Multi-Age Rule
> When a division spans multiple age groups (e.g., "U13/U14"), always take the **lower** age group. This is a conservative choice — it's better to slightly underestimate age than overestimate.

For birth year to age group conversion, see [[Age Groups and Birth Years]].

---

## Gender Parsing Priority

Checked in this order — first match wins:

| Priority | Method            | Example                       | Output |
| -------- | ----------------- | ----------------------------- | ------ |
| 1        | Explicit keyword  | "Boys", "Girls", "Male", "Female" | Boys/Girls |
| 2        | Code prefix       | "B10", "G10"                  | Boys/Girls |
| 3        | Compact suffix    | "13UB", "11UG"                | Boys/Girls |
| 4        | Event name        | "Girls Cup 2026"              | Girls  |
| 5        | Sibling inference | 95%+ of division is same gender | Apply to remaining |
| 6        | Team name pattern | "FC Girls U14"                | Girls  |

> [!info] Sibling Inference
> If 95%+ of games in a division already have the same gender assigned, the remaining null-gender games inherit that gender. This handles cases where most games in "ECNL Girls U15" got parsed correctly but a few entries had non-standard formatting.

---

## Round Detection Order

> [!warning] Order Is Critical
> Rounds are checked in THIS specific order to prevent substring conflicts. Checking "Final" before "Semi-Final" would incorrectly classify semi-finals as finals.

| Priority | Pattern Match       | Normalized Value  |
| -------- | ------------------- | ----------------- |
| 1        | Pool Play / Group   | `group`           |
| 2        | Semi-Final          | `semifinal`       |
| 3        | Quarter-Final       | `quarterfinal`    |
| 4        | Round of 16         | `round_of_16`     |
| 5        | Round of 8          | `quarterfinal`    |
| 6        | Round of 4          | `semifinal`       |
| 7        | 3rd Place           | `3rd_place`       |
| 8        | Final               | `final`           |
| 9        | Bracket / Knockout  | `knockout`        |
| 10       | Default (no match)  | `group`           |

Note: "Round of 8" maps to `quarterfinal` and "Round of 4" maps to `semifinal` — these are semantic equivalents.

---

## Season Assignment

The season spans an academic year:

| Game Date Range | Season Assigned |
| --------------- | --------------- |
| Aug 2025 - Dec 2025 | `2025-2026` |
| Jan 2026 - Jul 2026 | `2025-2026` |
| Aug 2026 - Dec 2026 | `2026-2027` |

**Rule:** Aug-Dec → `"YYYY-YYYY+1"`, Jan-Jul → `"YYYY-1-YYYY"`

---

## Score Parsing (GotSport HTML)

Used by [[GotSport HTML Scraper]]:

| Format                  | Example           | Home | Away | Notes          |
| ----------------------- | ----------------- | ---- | ---- | -------------- |
| Normal                  | "3 - 1"           | 3    | 1    | Standard       |
| Penalties               | "0 - 0PKS: 4 - 3"| 0    | 0    | PKS recorded separately |
| In progress             | "1 - 0*"          | 1    | 0    | Strip asterisk |

---

## Related Notes
- [[Data Pipeline]] — Steps 2-5 use these rules
- [[Age Groups and Birth Years]] — birth year conversion table
- [[GotSport HTML Scraper]] — score parsing specifics
- [[TGS AthleteOne]] — birth year division format (B2011, G2013)
- [[Data Quality]] — completeness metrics post-parsing
- [[Scope Rules]] — what age groups are in scope (U7-U19)
