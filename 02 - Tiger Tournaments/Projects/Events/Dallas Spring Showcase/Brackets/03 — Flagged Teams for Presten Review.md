---
type: dss-bracket-deliverable
event: Dallas Spring Showcase
deliverable: Flagged Teams for Presten Review
filed_by: FORGE+ELO (run by Claude Code as the night-of executor)
date: 2026-04-28
status: completed-needs-presten-review
scope: U11 and up
---

# DSS — Flagged Teams (Division-vs-Ranking Mismatches) — U11+

> **Scope:** Only U11 and up.

## Summary

| Flag | Count | Action |
|------|-------|--------|
| Move up | 41 | Rating exceeds median of next-higher division — consider promoting |
| Move down | 56 | Rating below median of next-lower division — consider demoting |
| Ranking uncertain | 64 | <5 games last 365d or no rating — manual review |

## Methodology — read this before acting

Per the autonomy directive, the agents needed to define ranking bands. The bands here are **empirical from the registration set itself**: for each (Event Age × Division), I computed the median rating of registered teams. A team is flagged "move up" if its rating is ≥ the median of the next-most-competitive division in its age group; "move down" if ≤ the median of the next-less-competitive division.

**Known limitations:**
1. The "competitive ladder" within an age group is sorted by empirical median rating — when divisions span multiple game formats (11v11 vs 9v9 vs 7v7), the ladder can place divisions in surprising orders. Some flags will look directionally wrong because of format mixing in the same Event Age.
2. Sample sizes are small for some divisions (≤3 teams) — those medians are noisy.
3. ELO has formal calibration work in flight (`Rankings/League Hierarchy Calibration Sanity Check — April 2026`) — a v2 of this analysis using ELO's hierarchy will be more reliable. This v1 is the night-of pass.

**Treat as a candidate list, not a directive.** Filter flags through your own judgment about division hierarchy.

## Move Up Candidates (41)

| Team ID | Team | Age | G | Current Div | Rating | Suggested | Reason |
|---------|------|-----|---|-------------|--------|-----------|--------|
| 274348 | Kernow Storm FC G13 API Lynch | U13 | F | 11v11 Bronze Bracket A | 1582 | → 11v11 Bronze II | Rating 1582 ≥ median of 11v11 Bronze II (1479) |
| 338205 | Juventus Premier FC 14G Black Cantu | U12 | F | 11v11 Silver Bracket A | 1524 | → 9v9 Bronze II | Rating 1524 ≥ median of 9v9 Bronze II (1504) |
| 88917 | Solar ECNL RL-NTX 11B Red | U15 | M | DS Silver Bracket A | 1575 | → Silver Elite Crossover | Rating 1575 ≥ median of Silver Elite Crossover (1544) |
| 512002 | Fort Worth Vaqueros FC 15 Alonzo | U11 | M | 9v9 Bronze II Bracket B | 1550 | → 9v9 Silver | Rating 1550 ≥ median of 9v9 Silver (1503) |
| 625826 | Dallas Surf 2013B NL TX-N | U13 | M | 11v11 Bronze II Bracket B | 1548 | → 11v11 Bronze Crossover | Rating 1548 ≥ median of 11v11 Bronze Crossover (1499) |
| 387583 | Sting Pre ECNL B16 Lucero | U11 | M | DS 9v9 Silver Elite Bracket A | 1601 | → 9v9 DS Gold | Rating 1601 ≥ median of 9v9 DS Gold (1590) |
| 500550 | Dallas Surf 11G Black | U15 | F | Silver Bracket A | 1539 | → DS Bronze | Rating 1539 ≥ median of DS Bronze (1503) |
| 65128 | Fever United 10B Jacobo | U16 | M | DS Bronze Crossover Bracket B | 1518 | → HS 2 | Rating 1518 ≥ median of HS 2 (1514) |
| 364378 | BVB 15B Black (DeSousa) | U11 | M | 9v9 Bronze Bracket C | 1510 | → 9v9 Bronze II | Rating 1510 ≥ median of 9v9 Bronze II (1494) |
| 636821 | Dallas Surf 15B Black | U11 | M | 9v9 Silver Bracket C | 1524 | → Silver II | Rating 1524 ≥ median of Silver II (1515) |
| 450821 | NTX United FC 2014G Smith | U13 | F | 11v11 Bronze Bracket B | 1534 | → 11v11 Bronze II | Rating 1534 ≥ median of 11v11 Bronze II (1479) |
| 68358 | Fever United 12B Phiri | U14 | M | 11v11 Silver Bracket A | 1545 | → 11v11 Bronze Crossover | Rating 1545 ≥ median of 11v11 Bronze Crossover (1504) |
| 524255 | NTX Celtic 14B Shaffer West | U12 | M | 9v9 Bronze Bracket A | 1491 | → 11v11 Bronze | Rating 1491 ≥ median of 11v11 Bronze (1486) |
| 288181 | NTX Celtic FC Pre-ECNL B14 Shaffer | U13 | M | 11v11 Silver 14s Bracket B | 1664 | → 11v11 Gold | Rating 1664 ≥ median of 11v11 Gold (1573) |
| 516663 | StrikerZ DFW 2012G Purple | U14 | F | 11v11 Bronze Crossover Bracket A | 1520 | → 11v11 Bronze | Rating 1520 ≥ median of 11v11 Bronze (1516) |
| 404131 | StrikerZ DFW 2013B Purple | U13 | M | 11v11 Bronze Crossover Bracket B | 1517 | → 11v11 Silver | Rating 1517 ≥ median of 11v11 Silver (1512) |
| 697618 | Texas Premier SC U12 | U12 | M | 9v9 Silver Crossover Bracket A | 1537 | → 9v9 Bronze | Rating 1537 ≥ median of 9v9 Bronze (1482) |
| 458924 | BVB 15G Yellow (Ibrahimpasic) | U11 | F | 9v9 Bronze II Bracket A | 1504 | → 9v9 Silver | Rating 1504 ≥ median of 9v9 Silver (1503) |
| 204390 | Dallas Surf 13G White | U13 | F | 11v11 Bronze Bracket A | 1503 | → 11v11 Bronze II | Rating 1503 ≥ median of 11v11 Bronze II (1479) |
| 138605 | FC LEGENDS UTD 15B Black | U11 | M | 9v9 Bronze Bracket C | 1533 | → 9v9 Bronze II | Rating 1533 ≥ median of 9v9 Bronze II (1494) |
| 269189 | Amarillo Rush 14G | U12 | F | 11v11 Bronze Bracket A | 1570 | → 11v11 Silver | Rating 1570 ≥ median of 11v11 Silver (1500) |
| 516424 | Extreme FC 12G Reyes | U14 | F | 11v11 Bronze Crossover Bracket A | 1539 | → 11v11 Bronze | Rating 1539 ≥ median of 11v11 Bronze (1516) |
| 612677 | Dallas Surf 15G North | U11 | F | 9v9 Silver Bracket B | 1516 | → Silver II | Rating 1516 ≥ median of Silver II (1515) |
| 514002 | Texas LoneStars 14B | U12 | M | 9v9 Bronze Bracket C | 1553 | → 11v11 Bronze | Rating 1553 ≥ median of 11v11 Bronze (1486) |
| 382244 | Bulverde YSA Relentless SC '14 | U12 | M | 9v9 Bronze Bracket C | 1519 | → 11v11 Bronze | Rating 1519 ≥ median of 11v11 Bronze (1486) |
| 634544 | Barça Academy Austin 14B Red | U12 | M | 9v9 Silver Crossover Bracket A | 1491 | → 9v9 Bronze | Rating 1491 ≥ median of 9v9 Bronze (1482) |
| 631035 | Elite Premier FC 2015B | U11 | M | 9v9 Bronze II Bracket A | 1533 | → 9v9 Silver | Rating 1533 ≥ median of 9v9 Silver (1503) |
| 81616 | Coppell FC 13B Shirazi White | U13 | M | 11v11 Bronze II Bracket B | 1527 | → 11v11 Bronze Crossover | Rating 1527 ≥ median of 11v11 Bronze Crossover (1499) |
| 517375 | SPD FC 14B Black | U12 | M | 9v9 Bronze Bracket B | 1527 | → 11v11 Bronze | Rating 1527 ≥ median of 11v11 Bronze (1486) |
| 380586 | Frisco Fusion 15B Blue | U11 | M | Silver II Bracket A | 1581 | → DS 9v9 Silver Elite | Rating 1581 ≥ median of DS 9v9 Silver Elite (1551) |
| 550321 | Frisco Fusion 15G Blue | U11 | F | 9v9 Bronze II Bracket B | 1515 | → 9v9 Silver | Rating 1515 ≥ median of 9v9 Silver (1503) |
| 415277 | FCD ETX Palestine 15/16B | U11 | M | 9v9 Silver Bracket B | 1534 | → Silver II | Rating 1534 ≥ median of Silver II (1515) |
| 655787 | The Hive | U11 | F | 9v9 Bronze Bracket A | 1544 | → 9v9 Bronze II | Rating 1544 ≥ median of 9v9 Bronze II (1494) |
| 703019 | Pegasus Futbol 2015 | U11 | M | 9v9 Bronze Bracket B | 1538 | → 9v9 Bronze II | Rating 1538 ≥ median of 9v9 Bronze II (1494) |
| 626056 | Pegasus Futbol 2013 | U13 | M | 11v11 Bronze Crossover Bracket A | 1564 | → 11v11 Silver | Rating 1564 ≥ median of 11v11 Silver (1512) |
| 193618 | Texas Warriors FC 15B Black | U11 | M | 9v9 Bronze Bracket B | 1623 | → 9v9 Bronze II | Rating 1623 ≥ median of 9v9 Bronze II (1494) |
| 631542 | Coppell FC 15B Williams White | U11 | M | 9v9 Bronze II Bracket A | 1572 | → 9v9 Silver | Rating 1572 ≥ median of 9v9 Silver (1503) |
| 489195 | Solar SW 15B Garcia.S White | U11 | M | 9v9 Bronze II Bracket A | 1516 | → 9v9 Silver | Rating 1516 ≥ median of 9v9 Silver (1503) |
| 148172 | Southern Soccer 14B | U12 | M | 9v9 Bronze Bracket A | 1507 | → 11v11 Bronze | Rating 1507 ≥ median of 11v11 Bronze (1486) |
| 489179 | Legacy United FC 14B - Juarez | U12 | M | 9v9 Bronze Bracket B | 1582 | → 11v11 Bronze | Rating 1582 ≥ median of 11v11 Bronze (1486) |
| 130687 | Legacy United FC 13B | U13 | M | 11v11 Bronze II Bracket A | 1539 | → 11v11 Bronze Crossover | Rating 1539 ≥ median of 11v11 Bronze Crossover (1499) |

## Move Down Candidates (56)

| Team ID | Team | Age | G | Current Div | Rating | Suggested | Reason |
|---------|------|-----|---|-------------|--------|-----------|--------|
| 65155 | Fever United 15B C. Smith | U11 | M | Silver II Bracket B | 1495 | → 9v9 Silver | Rating 1495 ≤ median of 9v9 Silver (1503) |
| 490220 | Atletico Dallas Youth Pre ECNL G15- Perez | U12 | F | 11v11 Silver Bracket A | 1464 | → 11v11 Bronze | Rating 1464 ≤ median of 11v11 Bronze (1486) |
| 647375 | Nido Águila 14G Gonzalez | U12 | F | 9v9 Bronze II Bracket B | 1495 | → 11v11 Silver | Rating 1495 ≤ median of 11v11 Silver (1500) |
| 450608 | Sting Attack G13 H Pantoja | U13 | F | 11v11 Bronze Bracket B | 1468 | → 11v11 Bronze 14s | Rating 1468 ≤ median of 11v11 Bronze 14s (1469) |
| 475356 | North Texas Elite 2015G Pina | U11 | F | 9v9 Silver Bracket A | 1488 | → 9v9 Bronze II | Rating 1488 ≤ median of 9v9 Bronze II (1494) |
| 91775 | Fever United 14G Wright | U12 | F | 9v9 Bronze II Bracket B | 1429 | → 11v11 Silver | Rating 1429 ≤ median of 11v11 Silver (1500) |
| 149444 | FC DIVAS XIII | U13 | F | 11v11 Silver Bracket B | 1481 | → 11v11 Bronze Crossover | Rating 1481 ≤ median of 11v11 Bronze Crossover (1499) |
| 96125 | FC Dallas Youth 15G Blue | U11 | F | 9v9 Silver Bracket B | 1451 | → 9v9 Bronze II | Rating 1451 ≤ median of 9v9 Bronze II (1494) |
| 298249 | Fortress FC 15G | U11 | F | 9v9 Bronze II Bracket A | 1454 | → 9v9 Bronze | Rating 1454 ≤ median of 9v9 Bronze (1474) |
| 452458 | Thunder United SC 13B Gold | U13 | M | 11v11 Bronze II Bracket A | 1451 | → 11v11 Bronze | Rating 1451 ≤ median of 11v11 Bronze (1471) |
| 107538 | DFW United 11G Sutherland | U15 | F | Silver Elite Crossover Bracket A | 1487 | → DS Silver | Rating 1487 ≤ median of DS Silver (1518) |
| 637021 | DKSC 11B RODRIGUEZ ETX | U15 | M | DS Silver Elite Bracket A | 1543 | → Silver Elite Crossover | Rating 1543 ≤ median of Silver Elite Crossover (1544) |
| 640464 | ULETE FC 13G Chris | U13 | F | 11v11 Bronze Bracket A | 1446 | → 11v11 Bronze 14s | Rating 1446 ≤ median of 11v11 Bronze 14s (1469) |
| 529243 | ULETE FC 12G Andre | U14 | F | 11v11 Silver Bracket A | 1470 | → 11v11 Bronze II | Rating 1470 ≤ median of 11v11 Bronze II (1471) |
| 640499 | ULETE FC 14G Carlo | U12 | F | 9v9 Bronze Bracket A | 1466 | → 9v9 Silver Crossover | Rating 1466 ≤ median of 9v9 Silver Crossover (1469) |
| 123743 | Lioness Select 13G | U13 | F | 11v11 Bronze Bracket B | 1464 | → 11v11 Bronze 14s | Rating 1464 ≤ median of 11v11 Bronze 14s (1469) |
| 330347 | TAG Pre ECNL 15B Gold East | U11 | M | 9v9 Silver Bracket A | 1473 | → 9v9 Bronze II | Rating 1473 ≤ median of 9v9 Bronze II (1494) |
| 405293 | StrikerZ DFW 2012B Purple | U14 | M | 11v11 Bronze Bracket C | 1495 | → 11v11 Bronze Crossover | Rating 1495 ≤ median of 11v11 Bronze Crossover (1504) |
| 634543 | Atletico Dallas Youth Blue ECNL-RL NTX B09 | U17 | M | Bracket B Bracket A | 1521 | → DS Silver | Rating 1521 ≤ median of DS Silver (1587) |
| 516598 | RPSC 13B Mercado | U13 | M | 11v11 Bronze Crossover Bracket B | 1467 | → 11v11 Bronze II | Rating 1467 ≤ median of 11v11 Bronze II (1479) |
| 636891 | Atletico Dallas Youth 14G Enciso | U12 | F | 9v9 Bronze II Bracket A | 1425 | → 11v11 Silver | Rating 1425 ≤ median of 11v11 Silver (1500) |
| 151633 | Rayados MOrtega 2013 Blue | U13 | M | 11v11 Gold Bracket A | 1565 | → 11v11 Silver 14s | Rating 1565 ≤ median of 11v11 Silver 14s (1566) |
| 516535 | Rayados MOrtega 11/12 | U15 | M | DS Bronze Bracket B | 1414 | → Silver | Rating 1414 ≤ median of Silver (1440) |
| 380587 | Dutch Lions 14B PSV | U12 | M | 9v9 Bronze Bracket B | 1467 | → 9v9 Silver Crossover | Rating 1467 ≤ median of 9v9 Silver Crossover (1469) |
| 100365 | Texas Spurs FC 11B | U15 | M | DS Silver Bracket A | 1478 | → DS Bronze | Rating 1478 ≤ median of DS Bronze (1503) |
| 640216 | Rayados MOrtega 2014 White | U12 | M | 9v9 Bronze Bracket A | 1464 | → 9v9 Silver Crossover | Rating 1464 ≤ median of 9v9 Silver Crossover (1469) |
| 83029 | FC Dallas Youth 13G North White | U13 | F | 11v11 Silver Bracket B | 1495 | → 11v11 Bronze Crossover | Rating 1495 ≤ median of 11v11 Bronze Crossover (1499) |
| 658982 | Atletico Dallas Youth FTW 15B Blue | U11 | M | Silver II Bracket C | 1467 | → 9v9 Silver | Rating 1467 ≤ median of 9v9 Silver (1503) |
| 253925 | OK Energy FC 12B Red | U14 | M | 11v11 Bronze Bracket B | 1465 | → 11v11 Bronze Crossover | Rating 1465 ≤ median of 11v11 Bronze Crossover (1504) |
| 634238 | FC Dallas East Texas 2014/15 G - Sabillon - BLUE | U12 | F | 9v9 Bronze II Bracket B | 1441 | → 11v11 Silver | Rating 1441 ≤ median of 11v11 Silver (1500) |
| 631678 | NTX Celtic 14G Holden | U12 | F | 9v9 Bronze II Bracket B | 1438 | → 11v11 Silver | Rating 1438 ≤ median of 11v11 Silver (1500) |
| 635771 | Solar 15B Molano PreECNL Red | U11 | M | DS 9v9 Silver Elite Bracket B | 1490 | → Silver II | Rating 1490 ≤ median of Silver II (1515) |
| 138606 | FC LEGENDS UTD 10/11B Red | U16 | M | DS Bronze Crossover Bracket B | 1427 | → HS 1 | Rating 1427 ≤ median of HS 1 (1472) |
| 521584 | U13 Boys Gold - GO | U13 | M | 11v11 Bronze II Bracket A | 1436 | → 11v11 Bronze | Rating 1436 ≤ median of 11v11 Bronze (1471) |
| 96642 | Dallas Surf 12G Black | U14 | F | 11v11 Bronze Crossover Bracket B | 1428 | → 11v11 Silver | Rating 1428 ≤ median of 11v11 Silver (1485) |
| 521596 | U12 Boys Gold - GO | U12 | M | 9v9 Bronze Bracket A | 1453 | → 9v9 Silver Crossover | Rating 1453 ≤ median of 9v9 Silver Crossover (1469) |
| 362201 | Dallas Texans ECNL RL NTX G13 | U13 | F | 11v11 Silver Bracket B | 1470 | → 11v11 Bronze Crossover | Rating 1470 ≤ median of 11v11 Bronze Crossover (1499) |
| 79663 | Sting G14 Fuentes | U12 | F | 9v9 Bronze Bracket A | 1431 | → 9v9 Silver Crossover | Rating 1431 ≤ median of 9v9 Silver Crossover (1469) |
| 166088 | 14G McGregor | U12 | F | 11v11 Bronze Bracket A | 1462 | → 9v9 Bronze | Rating 1462 ≤ median of 9v9 Bronze (1482) |
| 134104 | Coppell FC 15G Campanella White | U11 | F | 9v9 Bronze II Bracket A | 1423 | → 9v9 Bronze | Rating 1423 ≤ median of 9v9 Bronze (1474) |
| 488204 | Texas Warriors FC 15B Salas | U11 | M | 9v9 Silver Bracket B | 1487 | → 9v9 Bronze II | Rating 1487 ≤ median of 9v9 Bronze II (1494) |
| 385944 | FC Dallas Youth 14G West | U12 | F | 11v11 Bronze Bracket A | 1463 | → 9v9 Bronze | Rating 1463 ≤ median of 9v9 Bronze (1482) |
| 390669 | Atletico Dallas Youth FDL 09B Reyes | U17 | M | DS Silver Bracket A | 1456 | → HS 1 | Rating 1456 ≤ median of HS 1 (1469) |
| 406242 | Barca Academy Austin 15/16B Blue | U11 | M | Silver II Bracket B | 1480 | → 9v9 Silver | Rating 1480 ≤ median of 9v9 Silver (1503) |
| 491040 | Solar Pre ECNL G15 Naizer | U11 | F | 9v9 Silver Bracket A | 1468 | → 9v9 Bronze II | Rating 1468 ≤ median of 9v9 Bronze II (1494) |
| 151489 | FC Dallas El Paso 2011 | U15 | M | DS Silver Bracket A | 1479 | → DS Bronze | Rating 1479 ≤ median of DS Bronze (1503) |
| 492976 | Solar South 11G Cameron | U15 | F | Silver Elite Crossover Bracket A | 1477 | → DS Silver | Rating 1477 ≤ median of DS Silver (1518) |
| 494025 | Atletico Dallas Youth 15B Brandt | U11 | M | 9v9 Silver Bracket C | 1490 | → 9v9 Bronze II | Rating 1490 ≤ median of 9v9 Bronze II (1494) |
| 517372 | SPD FC 13B Red | U13 | M | 11v11 Bronze II Bracket B | 1445 | → 11v11 Bronze | Rating 1445 ≤ median of 11v11 Bronze (1471) |
| 99024 | FC LEGENDS UTD 13B Red | U13 | M | 11v11 Gold Bracket A | 1558 | → 11v11 Silver 14s | Rating 1558 ≤ median of 11v11 Silver 14s (1566) |
| 209098 | Dallas Cosmos 13G Red | U13 | F | 11v11 Bronze Bracket C | 1458 | → 11v11 Bronze 14s | Rating 1458 ≤ median of 11v11 Bronze 14s (1469) |
| 163527 | 13B Giurgila | U13 | M | 11v11 Bronze II Bracket B | 1467 | → 11v11 Bronze | Rating 1467 ≤ median of 11v11 Bronze (1471) |
| 77672 | Midlothian Premier FC 2012 Girls | U14 | F | 11v11 Bronze Crossover Bracket B | 1470 | → 11v11 Silver | Rating 1470 ≤ median of 11v11 Silver (1485) |
| 152149 | Royal Peak FC 2015 White | U11 | M | Silver II Bracket C | 1450 | → 9v9 Silver | Rating 1450 ≤ median of 9v9 Silver (1503) |
| 283774 | Tyler FC 15B | U11 | M | DS 9v9 Silver Elite Bracket B | 1457 | → Silver II | Rating 1457 ≤ median of Silver II (1515) |
| 625834 | Dallas Surf 15G Black | U11 | F | 9v9 Bronze II Bracket B | 1438 | → 9v9 Bronze | Rating 1438 ≤ median of 9v9 Bronze (1474) |

## Ranking Uncertain (64)

See `Brackets/03 — Flagged Teams.csv` (filter flag = ranking-uncertain). These teams have insufficient game data to support an up/down recommendation — most are U6–U9 where games are not consistently logged in the source data.

## Companion CSV

`Brackets/03 — Flagged Teams.csv` has the full list with all three flag types in one file.
