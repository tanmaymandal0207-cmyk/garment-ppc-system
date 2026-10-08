# Module 2: Machine capacity planner

`workbooks/PPC_Machine_Capacity_Planner_SAMPLE.xlsx`

## Sheet map

| Tab | Visible | Purpose |
|---|---|---|
| `GUIDE` | ✓ | Built-in manual: quick start, input cells, routines, self-checks, colour conventions, known gaps |
| `MD VIEW` | ✓ | Monthly decision view: installed, month-peak requirement, idle, utilisation, machines to **move**, machines to **buy**, days the plan can't run, and a group × day surplus/short grid |
| `DASHBOARD` | ✓ | Day / month-end / month-peak view: KPI tiles, status per machine group, top 6 bottlenecks, idle capacity, line loading, garment drill-down, peak-demand block |
| `LINE PLAN` | ✓ | The plan grid (26 lines × up to 200 days), a resolved grid, changeover flags, thermal-variant inputs, style list and colour legend |
| `MC MASTER` | ✓ | Installed machines: 26 groups in 14 categories (920 machines) |
| `EMBELISHMENT` | ✓ | Machine recipe per style: 25 operations × 42 styles, total machines and output per day |
| `MC MAP` | ✓ | The workshop: joins, alias table, style × machine matrix, line loading, month view |
| `DEMAND BY DATE` | hidden | Daily engine and working calendar |
| `WORKINGS` | hidden | Helpers: picked date or month, peak day, move/buy arithmetic |
| `PPC`, `MACHINE MASTER` | hidden | Archives of the original painted board and asset register (labelled *SUPERSEDED*; nothing reads them) |

## Data flow

```
LINE PLAN  B6:GS31   typed style per line per day (dropdown = HE5:HE47)
   │  B36:GS61        resolved style: THERMAL → variant named in GY36:GY61 (via MC MAP alias table)
   ▼
MC MAP  K5:M28        alias: planner's text → recipe column → category
MC MAP  A33:AG80      STYLE × MACHINE MATRIX (48 styles × 26 groups) + cross-foot check
MC MAP  CZ4:EV30      same matrix transposed, read by the daily engine
   ▼
DEMAND BY DATE        per day: lines per style (CA:DV) × matrix → demand per group (B:AA)
   ▼
DASHBOARD / MD VIEW   demand vs MC MASTER installed → SHORT / TIGHT / IDLE / OK, move / buy
```

## Key formulas

**Plan dates.** The horizon comes from two cells:

```excel
LINE PLAN!B5  =IF(COLUMN()-1>$B$3, 0, $B$2+COLUMN()-2)        ' B2 = plan start, B3 = days in plan (≤ 200)
```

**Resolve thermal.** `THERMAL` is a category. The line's named variant is substituted:

```excel
LINE PLAN!B36 =IF(B6="","",IF(B6<>"THERMAL",B6,IF($GW36="","THERMAL - NOT SPECIFIED",$GW36)))
```

**Style × machine matrix.** A recipe re-grouped from operations into machine groups:

```excel
MC MAP!B33 =IFERROR(SUMIF($B$5:$B$29, B$32,
              INDEX(EMBELISHMENT!$B$3:$AQ$27, 0, MATCH($AE33, EMBELISHMENT!$B$1:$AQ$1, 0))), 0)
MC MAP!AD33 =IF(N(AC33)=0,"n/a",IF(ROUND(AB33-AC33,1)=0,"OK",ROUND(AB33-AC33,1)))   ' cross-foot vs recipe total
```

**Daily demand per machine group.** A count of lines per style, dot-multiplied by the matrix:

```excel
DEMAND BY DATE!B4  =SUMPRODUCT($CA4:$DV4, INDEX('MC MAP'!$DA$5:$EV$30, 1, 0))
DEMAND BY DATE!AX4 =IF(OR($AB4=0,UPPER($BS4)<>"Y"),0,SUMPRODUCT(--($B4:$AA4>$B$205:$AA$205)))   ' groups over capacity
```

**Group status on the dashboard:**

```excel
DASHBOARD!F12 =IF($D12=0,"IDLE",IF($E12<0,"SHORT",IF($E12/$C12<=0.1,"TIGHT","OK")))
```

`TIGHT` means less than 10% spare, so a single breakdown stops a line.

**Move or buy.** A shortage is first covered by idle machines in the same category. Only the remainder is a buy:

```excel
DASHBOARD!H12 =IF($E12>=0, category,
                IF(spare_in_category<=0, category&" — no spare, BUY "&need,
                IF(still_short<=0, category&" — MOVE "&spare, category&" — MOVE "&spare&", BUY "&still_short)))
```

## Self-checks

| Where | What it tells you |
|---|---|
| `LINE PLAN!HA2` | Green when every planned cell can be priced. Otherwise it counts the cells whose style has no recipe, which would cost zero machines |
| `MC MAP!AD33:AD80` | `OK` / `n/a` per style. A number means the re-grouped total disagrees with the recipe sheet |
| `MC MAP!AG33:AG80` | Pink `NEEDS INPUT` = garment type not set (affects only the garment drill-down) |
| `DEMAND BY DATE!AW` | Lines with `THERMAL` days but no variant named |
| `DEMAND BY DATE!AX` | Machine groups over capacity on that day |

## Inputs (all other cells are locked)

`LINE PLAN` B2, B3, B6:GS31, GY36:GY61 · `MC MASTER` A3:C28 · `EMBELISHMENT` · `MC MAP` AG33:AG80, AJ33:AJ43, K5:M28, B120 · `DASHBOARD` C4 / D4 / E4 / K71 · `MD VIEW` D4 · `DEMAND BY DATE` BS (working-day calendar).

## Limits

- The horizon is pre-built to 200 days.
- Month views cover the first four calendar months from the plan start.
- A new machine type needs the 26-group width of `MC MAP` and `DEMAND BY DATE` to be extended.
