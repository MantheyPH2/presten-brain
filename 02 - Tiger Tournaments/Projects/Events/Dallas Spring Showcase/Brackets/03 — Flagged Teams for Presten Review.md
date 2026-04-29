---
type: dss-bracket-deliverable
event: Dallas Spring Showcase
deliverable: Flagged Teams for Presten Review
filed_by: FORGE+ELO (run by Claude Code as the night-of executor)
date: 2026-04-28
status: completed-needs-presten-review
---

# DSS — Flagged Teams (Division-vs-Ranking Mismatches)

## Summary

| Flag | Count | Action |
|------|-------|--------|
| Move up | 67 | Rating exceeds median of next-higher division — consider promoting |
| Move down | 83 | Rating below median of next-lower division — consider demoting |
| Ranking uncertain | 133 | <5 games last 365d or no rating — manual review |

## Methodology — read this before acting

Per the autonomy directive, the agents needed to define ranking bands. The bands here are **empirical from the registration set itself**: for each (Event Age × Division), I computed the median rating of registered teams. A team is flagged "move up" if its rating is ≥ the median of the next-most-competitive division in its age group; "move down" if ≤ the median of the next-less-competitive division.

**Known limitations:**
1. The "competitive ladder" within an age group is sorted by empirical median rating — when divisions span multiple game formats (11v11 vs 9v9 vs 7v7), the ladder can place divisions in surprising orders. Some flags will look directionally wrong because of format mixing in the same Event Age.
2. Sample sizes are small for some divisions (≤3 teams) — those medians are noisy.
3. ELO has formal calibration work in flight (`Rankings/League Hierarchy Calibration Sanity Check — April 2026`) — a v2 of this analysis using ELO's hierarchy will be more reliable. This v1 is the night-of pass.

**Treat as a candidate list, not a directive.** Filter flags through your own judgment about division hierarchy.

## Move Up Candidates (67)

| Team ID | Team | Age | G | Current Div | Rating | Suggested | Reason |
|---------|------|-----|---|-------------|--------|-----------|--------|
| 274348 | Kernow Storm FC G13 API Lynch | U13 | F | 11v11 Bronze Bracket A | 1582 | → 11v11 Bronze II | Rating 1582 ≥ median of 11v11 Bronze II (1479) |
| 577503 | FC Dallas Youth 16B Palmisano | U10 | M | 9v9 16s Silver Bracket A | 1550 | → 9v9 17s | Rating 1550 ≥ median of 9v9 17s (1513) |
| 338205 | Juventus Premier FC 14G Black Cantu | U12 | F | 11v11 Silver Bracket A | 1524 | → 9v9 Bronze II | Rating 1524 ≥ median of 9v9 Bronze II (1504) |
| 88917 | Solar ECNL RL-NTX 11B Red | U15 | M | DS Silver Bracket A | 1575 | → Silver Elite Crossover | Rating 1575 ≥ median of Silver Elite Crossover (1544) |
| 512002 | Fort Worth Vaqueros FC 15 Alonzo | U11 | M | 9v9 Bronze II Bracket B | 1550 | → 9v9 Silver | Rating 1550 ≥ median of 9v9 Silver (1503) |
| 625826 | Dallas Surf 2013B NL TX-N | U13 | M | 11v11 Bronze II Bracket B | 1548 | → 11v11 Bronze Crossover | Rating 1548 ≥ median of 11v11 Bronze Crossover (1499) |
| 387583 | Sting Pre ECNL B16 Lucero | U11 | M | DS 9v9 Silver Elite Bracket A | 1601 | → 9v9 DS Gold | Rating 1601 ≥ median of 9v9 DS Gold (1590) |
| 500550 | Dallas Surf 11G Black | U15 | F | Silver Bracket A | 1539 | → DS Bronze | Rating 1539 ≥ median of DS Bronze (1503) |
| 590352 | FC Dallas Youth 17B Central Mata | U9 | M | 7v7 Silver Bracket A | 1539 | → 7v7 Dallas Showcase Super Group 17s | Rating 1539 ≥ median of 7v7 Dallas Showcase Super Group 17s (1532) |
| 686296 | FC Dallas Youth 18B Mata Central | U8 | M | 4v4 Silver Bracket A | 1501 | → 7v7 Bronze | Rating 1501 ≥ median of 7v7 Bronze (1474) |
| 651878 | ULETE FC 18G Jasmine | U8 | F | 4v4 Bronze Bracket A | 1463 | → 4v4 Silver | Rating 1463 ≥ median of 4v4 Silver (1455) |
| 65128 | Fever United 10B Jacobo | U16 | M | DS Bronze Crossover Bracket B | 1518 | → HS 2 | Rating 1518 ≥ median of HS 2 (1514) |
| 364378 | BVB 15B Black (DeSousa) | U11 | M | 9v9 Bronze Bracket C | 1510 | → 9v9 Bronze II | Rating 1510 ≥ median of 9v9 Bronze II (1494) |
| 541887 | AlphaForms'17G | U9 | F | 7v7 Dallas Showcase Super Group 17s Bracket A | 1578 | → 18s | Rating 1578 ≥ median of 18s (1534) |
| 386019 | FC Dallas Youth Pre ECNL G16 Avant West | U10 | F | 9v9 Silver Bracket B | 1512 | → 9v9 16s Silver | Rating 1512 ≥ median of 9v9 16s Silver (1510) |
| 579139 | FC Guardians 2016 | U10 | M | 9v9 Bronze Bracket A | 1546 | → 9v9 Silver | Rating 1546 ≥ median of 9v9 Silver (1509) |
| 636821 | Dallas Surf 15B Black | U11 | M | 9v9 Silver Bracket C | 1524 | → Silver II | Rating 1524 ≥ median of Silver II (1515) |
| 122868 | Dortmund 18B | U8 | M | 7v7 Bronze Bracket B | 1494 | → 7v7 Silver | Rating 1494 ≥ median of 7v7 Silver (1490) |
| 450821 | NTX United FC 2014G Smith | U13 | F | 11v11 Bronze Bracket B | 1534 | → 11v11 Bronze II | Rating 1534 ≥ median of 11v11 Bronze II (1479) |
| 68358 | Fever United 12B Phiri | U14 | M | 11v11 Silver Bracket A | 1545 | → 11v11 Bronze Crossover | Rating 1545 ≥ median of 11v11 Bronze Crossover (1504) |
| 579647 | Sting B18 Guess | U8 | M | 7v7 18s Silver Bracket B | 1638 | → Dallas Showcase Super Group 7v7 18s | Rating 1638 ≥ median of Dallas Showcase Super Group 7v7 18s (1580) |
| 441539 | Sting Attack 17G D Sarmiento | U10 | F | 9v9 DS Gold Bracket B | 1603 | → 9v9 Gold | Rating 1603 ≥ median of 9v9 Gold (1588) |
| 513849 | ETX United 16G - Swiney | U10 | F | 9v9 Silver Bracket B | 1557 | → 9v9 16s Silver | Rating 1557 ≥ median of 9v9 16s Silver (1510) |
| 524255 | NTX Celtic 14B Shaffer West | U12 | M | 9v9 Bronze Bracket A | 1491 | → 11v11 Bronze | Rating 1491 ≥ median of 11v11 Bronze (1486) |
| 288181 | NTX Celtic FC Pre-ECNL B14 Shaffer | U13 | M | 11v11 Silver 14s Bracket B | 1664 | → 11v11 Gold | Rating 1664 ≥ median of 11v11 Gold (1573) |
| 576833 | ULETE FC 17G RAB | U9 | F | 7v7 Bronze II Crossover Bracket A | 1476 | → 7v7 Bronze Crossover | Rating 1476 ≥ median of 7v7 Bronze Crossover (1474) |
| 553844 | RPSC 16B Mercado_Blue | U10 | M | 9v9 16s Silver Elite Bracket A | 1555 | → 9v9 Bronze II | Rating 1555 ≥ median of 9v9 Bronze II (1534) |
| 478937 | Atletico Dallas Youth Pre-ECNL G16 Meighen | U10 | F | 9v9 DS Gold Bracket A | 1647 | → 9v9 Gold | Rating 1647 ≥ median of 9v9 Gold (1588) |
| 516663 | StrikerZ DFW 2012G Purple | U14 | F | 11v11 Bronze Crossover Bracket A | 1520 | → 11v11 Bronze | Rating 1520 ≥ median of 11v11 Bronze (1516) |
| 404131 | StrikerZ DFW 2013B Purple | U13 | M | 11v11 Bronze Crossover Bracket B | 1517 | → 11v11 Silver | Rating 1517 ≥ median of 11v11 Silver (1512) |
| 697618 | Texas Premier SC U12 | U12 | M | 9v9 Silver Crossover Bracket A | 1537 | → 9v9 Bronze | Rating 1537 ≥ median of 9v9 Bronze (1482) |
| 458924 | BVB 15G Yellow (Ibrahimpasic) | U11 | F | 9v9 Bronze II Bracket A | 1504 | → 9v9 Silver | Rating 1504 ≥ median of 9v9 Silver (1503) |
| 146891 | DFW United SC 16B - BLUE | U10 | M | 9v9 16s Silver Elite Bracket B | 1558 | → 9v9 Bronze II | Rating 1558 ≥ median of 9v9 Bronze II (1534) |
| 204390 | Dallas Surf 13G White | U13 | F | 11v11 Bronze Bracket A | 1503 | → 11v11 Bronze II | Rating 1503 ≥ median of 11v11 Bronze II (1479) |
| 138605 | FC LEGENDS UTD 15B Black | U11 | M | 9v9 Bronze Bracket C | 1533 | → 9v9 Bronze II | Rating 1533 ≥ median of 9v9 Bronze II (1494) |
| 701410 | Rayos 2019b Red | U7 | M | Bronze 4v4 Bracket B | 1510 | → Silver 4v4 | Rating 1510 ≥ median of Silver 4v4 (1495) |
| 579593 | Sting Attack G17 D Sarmiento White | U9 | F | 7v7 Dallas Showcase Super Group 17s Bracket A | 1599 | → 18s | Rating 1599 ≥ median of 18s (1534) |
| 269189 | Amarillo Rush 14G | U12 | F | 11v11 Bronze Bracket A | 1570 | → 11v11 Silver | Rating 1570 ≥ median of 11v11 Silver (1500) |
| 646179 | Sting B17 Ortiz | U10 | M | 9v9 17s Bracket B | 1602 | → 9v9 16s Silver Elite | Rating 1602 ≥ median of 9v9 16s Silver Elite (1527) |
| 501618 | FC LEGENDS UTD 16B Black | U10 | M | 9v9 Bronze II Crossover Bracket B | 1556 | → 9v9 Bronze | Rating 1556 ≥ median of 9v9 Bronze (1468) |
| 516424 | Extreme FC 12G Reyes | U14 | F | 11v11 Bronze Crossover Bracket A | 1539 | → 11v11 Bronze | Rating 1539 ≥ median of 11v11 Bronze (1516) |
| 658373 | ISW Academy 18 Munoz | U8 | M | 7v7 Bronze Bracket B | 1637 | → 7v7 Silver | Rating 1637 ≥ median of 7v7 Silver (1490) |
| 612677 | Dallas Surf 15G North | U11 | F | 9v9 Silver Bracket B | 1516 | → Silver II | Rating 1516 ≥ median of Silver II (1515) |
| 514002 | Texas LoneStars 14B | U12 | M | 9v9 Bronze Bracket C | 1553 | → 11v11 Bronze | Rating 1553 ≥ median of 11v11 Bronze (1486) |
| 382244 | Bulverde YSA Relentless SC '14 | U12 | M | 9v9 Bronze Bracket C | 1519 | → 11v11 Bronze | Rating 1519 ≥ median of 11v11 Bronze (1486) |
| 634544 | Barça Academy Austin 14B Red | U12 | M | 9v9 Silver Crossover Bracket A | 1491 | → 9v9 Bronze | Rating 1491 ≥ median of 9v9 Bronze (1482) |
| 631035 | Elite Premier FC 2015B | U11 | M | 9v9 Bronze II Bracket A | 1533 | → 9v9 Silver | Rating 1533 ≥ median of 9v9 Silver (1503) |
| 484573 | Atletico Dallas Youth G17 Meighen | U10 | F | 9v9 17s Bracket A | 1638 | → 9v9 16s Silver Elite | Rating 1638 ≥ median of 9v9 16s Silver Elite (1527) |
| 654833 | BVB 16B White (Kennedy) | U10 | M | 9v9 16s Silver Bracket B | 1579 | → 9v9 17s | Rating 1579 ≥ median of 9v9 17s (1513) |
| 643624 | OKC 405 FC 2016 G - Elite | U10 | F | 9v9 DS Gold Bracket B | 1605 | → 9v9 Gold | Rating 1605 ≥ median of 9v9 Gold (1588) |
| 81616 | Coppell FC 13B Shirazi White | U13 | M | 11v11 Bronze II Bracket B | 1527 | → 11v11 Bronze Crossover | Rating 1527 ≥ median of 11v11 Bronze Crossover (1499) |
| 517375 | SPD FC 14B Black | U12 | M | 9v9 Bronze Bracket B | 1527 | → 11v11 Bronze | Rating 1527 ≥ median of 11v11 Bronze (1486) |
| 380586 | Frisco Fusion 15B Blue | U11 | M | Silver II Bracket A | 1581 | → DS 9v9 Silver Elite | Rating 1581 ≥ median of DS 9v9 Silver Elite (1551) |
| 550321 | Frisco Fusion 15G Blue | U11 | F | 9v9 Bronze II Bracket B | 1515 | → 9v9 Silver | Rating 1515 ≥ median of 9v9 Silver (1503) |
| 415277 | FCD ETX Palestine 15/16B | U11 | M | 9v9 Silver Bracket B | 1534 | → Silver II | Rating 1534 ≥ median of Silver II (1515) |
| 655787 | The Hive | U11 | F | 9v9 Bronze Bracket A | 1544 | → 9v9 Bronze II | Rating 1544 ≥ median of 9v9 Bronze II (1494) |
| 703019 | Pegasus Futbol 2015 | U11 | M | 9v9 Bronze Bracket B | 1538 | → 9v9 Bronze II | Rating 1538 ≥ median of 9v9 Bronze II (1494) |
| 626056 | Pegasus Futbol 2013 | U13 | M | 11v11 Bronze Crossover Bracket A | 1564 | → 11v11 Silver | Rating 1564 ≥ median of 11v11 Silver (1512) |
| 419884 | 17B McGregor (Black) | U9 | M | 7v7 Bronze II Bracket A | 1497 | → 7v7 Bronze II Crossover | Rating 1497 ≥ median of 7v7 Bronze II Crossover (1436) |
| 193618 | Texas Warriors FC 15B Black | U11 | M | 9v9 Bronze Bracket B | 1623 | → 9v9 Bronze II | Rating 1623 ≥ median of 9v9 Bronze II (1494) |
| 550627 | Liverpool FC Dallas 2016B | U10 | M | 9v9 Bronze II Crossover Bracket A | 1468 | → 9v9 Bronze | Rating 1468 ≥ median of 9v9 Bronze (1468) |
| 631542 | Coppell FC 15B Williams White | U11 | M | 9v9 Bronze II Bracket A | 1572 | → 9v9 Silver | Rating 1572 ≥ median of 9v9 Silver (1503) |
| 523891 | Dallas Texans 2017 Boys Red South | U10 | M | 9v9 Bronze II Crossover Bracket A | 1550 | → 9v9 Bronze | Rating 1550 ≥ median of 9v9 Bronze (1468) |
| 489195 | Solar SW 15B Garcia.S White | U11 | M | 9v9 Bronze II Bracket A | 1516 | → 9v9 Silver | Rating 1516 ≥ median of 9v9 Silver (1503) |
| 148172 | Southern Soccer 14B | U12 | M | 9v9 Bronze Bracket A | 1507 | → 11v11 Bronze | Rating 1507 ≥ median of 11v11 Bronze (1486) |
| 489179 | Legacy United FC 14B - Juarez | U12 | M | 9v9 Bronze Bracket B | 1582 | → 11v11 Bronze | Rating 1582 ≥ median of 11v11 Bronze (1486) |
| 130687 | Legacy United FC 13B | U13 | M | 11v11 Bronze II Bracket A | 1539 | → 11v11 Bronze Crossover | Rating 1539 ≥ median of 11v11 Bronze Crossover (1499) |

## Move Down Candidates (83)

| Team ID | Team | Age | G | Current Div | Rating | Suggested | Reason |
|---------|------|-----|---|-------------|--------|-----------|--------|
| 65155 | Fever United 15B C. Smith | U11 | M | Silver II Bracket B | 1495 | → 9v9 Silver | Rating 1495 ≤ median of 9v9 Silver (1503) |
| 490220 | Atletico Dallas Youth Pre ECNL G15- Perez | U12 | F | 11v11 Silver Bracket A | 1464 | → 11v11 Bronze | Rating 1464 ≤ median of 11v11 Bronze (1486) |
| 496328 | North Texas Elite 19B Pina | U7 | M | Gold 4v4 Bracket A | 1491 | → Silver 4v4 | Rating 1491 ≤ median of Silver 4v4 (1495) |
| 647375 | Nido Águila 14G Gonzalez | U12 | F | 9v9 Bronze II Bracket B | 1495 | → 11v11 Silver | Rating 1495 ≤ median of 11v11 Silver (1500) |
| 620522 | DKSC 16B RAMOS | U10 | M | 9v9 16s Silver Bracket A | 1497 | → 9v9 Silver | Rating 1497 ≤ median of 9v9 Silver (1509) |
| 450608 | Sting Attack G13 H Pantoja | U13 | F | 11v11 Bronze Bracket B | 1468 | → 11v11 Bronze 14s | Rating 1468 ≤ median of 11v11 Bronze 14s (1469) |
| 691814 | Atletico Dallas Youth 17G South Gray | U9 | F | 7v7 Bronze II Crossover Bracket A | 1426 | → 7v7 Bronze II | Rating 1426 ≤ median of 7v7 Bronze II (1434) |
| 475356 | North Texas Elite 2015G Pina | U11 | F | 9v9 Silver Bracket A | 1488 | → 9v9 Bronze II | Rating 1488 ≤ median of 9v9 Bronze II (1494) |
| 380066 | Solar South 17G Guzman | U10 | F | 9v9 17s Bracket A | 1484 | → 9v9 16s Silver | Rating 1484 ≤ median of 9v9 16s Silver (1510) |
| 91775 | Fever United 14G Wright | U12 | F | 9v9 Bronze II Bracket B | 1429 | → 11v11 Silver | Rating 1429 ≤ median of 11v11 Silver (1500) |
| 441884 | Soccerplex United 17B Red | U9 | M | 7v7 Silver Bracket B | 1452 | → 7v7 Bronze Crossover | Rating 1452 ≤ median of 7v7 Bronze Crossover (1474) |
| 149444 | FC DIVAS XIII | U13 | F | 11v11 Silver Bracket B | 1481 | → 11v11 Bronze Crossover | Rating 1481 ≤ median of 11v11 Bronze Crossover (1499) |
| 96125 | FC Dallas Youth 15G Blue | U11 | F | 9v9 Silver Bracket B | 1451 | → 9v9 Bronze II | Rating 1451 ≤ median of 9v9 Bronze II (1494) |
| 233278 | DKSC 16B Pre-ECNL DE LA CRUZ | U10 | M | 9v9 Gold Bracket A | 1535 | → 9v9 DS Gold | Rating 1535 ≤ median of 9v9 DS Gold (1582) |
| 298249 | Fortress FC 15G | U11 | F | 9v9 Bronze II Bracket A | 1454 | → 9v9 Bronze | Rating 1454 ≤ median of 9v9 Bronze (1474) |
| 454411 | Revolution Premier SC 16G Blue | U10 | F | 9v9 Bronze II Bracket B | 1385 | → 9v9 16s Silver Elite | Rating 1385 ≤ median of 9v9 16s Silver Elite (1527) |
| 389719 | Texas Warriors FC 16G Red | U10 | F | 9v9 Bronze II Bracket A | 1440 | → 9v9 16s Silver Elite | Rating 1440 ≤ median of 9v9 16s Silver Elite (1527) |
| 452458 | Thunder United SC 13B Gold | U13 | M | 11v11 Bronze II Bracket A | 1451 | → 11v11 Bronze | Rating 1451 ≤ median of 11v11 Bronze (1471) |
| 107538 | DFW United 11G Sutherland | U15 | F | Silver Elite Crossover Bracket A | 1487 | → DS Silver | Rating 1487 ≤ median of DS Silver (1518) |
| 637021 | DKSC 11B RODRIGUEZ ETX | U15 | M | DS Silver Elite Bracket A | 1543 | → Silver Elite Crossover | Rating 1543 ≤ median of Silver Elite Crossover (1544) |
| 633580 | NTX Celtic 19B Monk West | U7 | M | Silver 4v4 Bracket A | 1484 | → Bronze 4v4 | Rating 1484 ≤ median of Bronze 4v4 (1485) |
| 356455 | Dallas Surf 17G East Blue | U10 | F | 9v9 17s Bracket A | 1493 | → 9v9 16s Silver | Rating 1493 ≤ median of 9v9 16s Silver (1510) |
| 693399 | Falls Town FC 17G | U9 | F | 7v7 Dallas Showcase Super Group 17s Bracket B | 1498 | → 7v7 Silver | Rating 1498 ≤ median of 7v7 Silver (1499) |
| 640464 | ULETE FC 13G Chris | U13 | F | 11v11 Bronze Bracket A | 1446 | → 11v11 Bronze 14s | Rating 1446 ≤ median of 11v11 Bronze 14s (1469) |
| 547188 | ULETE FC 17B Simon | U9 | M | 7v7 Bronze Crossover Bracket A | 1411 | → 7v7 Bronze II Crossover | Rating 1411 ≤ median of 7v7 Bronze II Crossover (1436) |
| 529243 | ULETE FC 12G Andre | U14 | F | 11v11 Silver Bracket A | 1470 | → 11v11 Bronze II | Rating 1470 ≤ median of 11v11 Bronze II (1471) |
| 640499 | ULETE FC 14G Carlo | U12 | F | 9v9 Bronze Bracket A | 1466 | → 9v9 Silver Crossover | Rating 1466 ≤ median of 9v9 Silver Crossover (1469) |
| 651942 | ULETE FC 19B Jordan | U7 | M | Bronze 4v4 Bracket B | 1421 | → 4v4 2019s | Rating 1421 ≤ median of 4v4 2019s (1476) |
| 442024 | BVB 16B Pre ECNL Brown | U10 | M | 9v9 16s Silver Bracket A | 1467 | → 9v9 Silver | Rating 1467 ≤ median of 9v9 Silver (1509) |
| 123743 | Lioness Select 13G | U13 | F | 11v11 Bronze Bracket B | 1464 | → 11v11 Bronze 14s | Rating 1464 ≤ median of 11v11 Bronze 14s (1469) |
| 330347 | TAG Pre ECNL 15B Gold East | U11 | M | 9v9 Silver Bracket A | 1473 | → 9v9 Bronze II | Rating 1473 ≤ median of 9v9 Bronze II (1494) |
| 405293 | StrikerZ DFW 2012B Purple | U14 | M | 11v11 Bronze Bracket C | 1495 | → 11v11 Bronze Crossover | Rating 1495 ≤ median of 11v11 Bronze Crossover (1504) |
| 634543 | Atletico Dallas Youth Blue ECNL-RL NTX B09 | U17 | M | Bracket B Bracket A | 1521 | → DS Silver | Rating 1521 ≤ median of DS Silver (1587) |
| 516598 | RPSC 13B Mercado | U13 | M | 11v11 Bronze Crossover Bracket B | 1467 | → 11v11 Bronze II | Rating 1467 ≤ median of 11v11 Bronze II (1479) |
| 549269 | RPSC 18B Mercado | U8 | M | 7v7 Bronze Bracket A | 1441 | → 4v4 Silver | Rating 1441 ≤ median of 4v4 Silver (1455) |
| 638843 | RPSC 19B Mercado_White | U7 | M | Bronze 4v4 Bracket A | 1412 | → 4v4 2019s | Rating 1412 ≤ median of 4v4 2019s (1476) |
| 616668 | Texas Premier SC 16B | U10 | M | 9v9 Bronze Bracket A | 1437 | → 9v9 Bronze II Crossover | Rating 1437 ≤ median of 9v9 Bronze II Crossover (1441) |
| 636891 | Atletico Dallas Youth 14G Enciso | U12 | F | 9v9 Bronze II Bracket A | 1425 | → 11v11 Silver | Rating 1425 ≤ median of 11v11 Silver (1500) |
| 151633 | Rayados MOrtega 2013 Blue | U13 | M | 11v11 Gold Bracket A | 1565 | → 11v11 Silver 14s | Rating 1565 ≤ median of 11v11 Silver 14s (1566) |
| 516535 | Rayados MOrtega 11/12 | U15 | M | DS Bronze Bracket B | 1414 | → Silver | Rating 1414 ≤ median of Silver (1440) |
| 380587 | Dutch Lions 14B PSV | U12 | M | 9v9 Bronze Bracket B | 1467 | → 9v9 Silver Crossover | Rating 1467 ≤ median of 9v9 Silver Crossover (1469) |
| 100365 | Texas Spurs FC 11B | U15 | M | DS Silver Bracket A | 1478 | → DS Bronze | Rating 1478 ≤ median of DS Bronze (1503) |
| 640216 | Rayados MOrtega 2014 White | U12 | M | 9v9 Bronze Bracket A | 1464 | → 9v9 Silver Crossover | Rating 1464 ≤ median of 9v9 Silver Crossover (1469) |
| 83029 | FC Dallas Youth 13G North White | U13 | F | 11v11 Silver Bracket B | 1495 | → 11v11 Bronze Crossover | Rating 1495 ≤ median of 11v11 Bronze Crossover (1499) |
| 450377 | Revolution Premier SC 18G Blue | U8 | F | 7v7 Silver Bracket A | 1451 | → 7v7 Bronze | Rating 1451 ≤ median of 7v7 Bronze (1474) |
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
| 661810 | Atletico Dallas Youth Central 18G Welge | U8 | F | 7v7 Silver Bracket A | 1455 | → 7v7 Bronze | Rating 1455 ≤ median of 7v7 Bronze (1474) |
| 272284 | Coppell FC 16B Reeck Red | U10 | M | 9v9 Gold Bracket A | 1493 | → 9v9 DS Gold | Rating 1493 ≤ median of 9v9 DS Gold (1582) |
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
| 279157 | Metro West Wolves 16B Thiem | U10 | M | 9v9 16s Silver Elite Bracket B | 1512 | → 9v9 17s | Rating 1512 ≤ median of 9v9 17s (1513) |
| 441885 | Soccerplex United 16B | U10 | M | 9v9 16s Silver Bracket B | 1473 | → 9v9 Silver | Rating 1473 ≤ median of 9v9 Silver (1509) |
| 152149 | Royal Peak FC 2015 White | U11 | M | Silver II Bracket C | 1450 | → 9v9 Silver | Rating 1450 ≤ median of 9v9 Silver (1503) |
| 521835 | Kernow Storm FC G16 Pre-ECNL Nelson | U10 | F | 9v9 DS Gold Bracket B | 1510 | → 9v9 Bronze II | Rating 1510 ≤ median of 9v9 Bronze II (1534) |
| 552488 | Solar 17G Jenkins | U9 | F | 7v7 Dallas Showcase Super Group 17s Bracket B | 1465 | → 7v7 Silver | Rating 1465 ≤ median of 7v7 Silver (1499) |
| 530905 | Soccerplex United 16/17G Red | U10 | F | 9v9 Silver Bracket B | 1428 | → 9v9 Bronze | Rating 1428 ≤ median of 9v9 Bronze (1468) |
| 645936 | Dallas Texans 2019 Girls East | U8 | F | 7v7 Silver Bracket B | 1456 | → 7v7 Bronze | Rating 1456 ≤ median of 7v7 Bronze (1474) |
| 283774 | Tyler FC 15B | U11 | M | DS 9v9 Silver Elite Bracket B | 1457 | → Silver II | Rating 1457 ≤ median of Silver II (1515) |
| 407081 | Tyler FC 18B | U9 | M | 18s Bracket A | 1508 | → 7v7 Dallas Showcase Super Group 17s | Rating 1508 ≤ median of 7v7 Dallas Showcase Super Group 17s (1532) |
| 625834 | Dallas Surf 15G Black | U11 | F | 9v9 Bronze II Bracket B | 1438 | → 9v9 Bronze | Rating 1438 ≤ median of 9v9 Bronze (1474) |

## Ranking Uncertain (133)

See `Brackets/03 — Flagged Teams.csv` (filter flag = ranking-uncertain). These teams have insufficient game data to support an up/down recommendation — most are U6–U9 where games are not consistently logged in the source data.

## Companion CSV

`Brackets/03 — Flagged Teams.csv` has the full list with all three flag types in one file.
