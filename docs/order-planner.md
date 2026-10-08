# Module 1: Order planner

`workbooks/PPC_Order_Planner_SAMPLE.xlsx`, sheets `Master` and `Holidays`.

**Blue cells** are inputs and **grey cells** are formulas. Rows 3–200 are pre-built. Working days exclude Sundays (`"0000001"` weekend mask) and the dates listed in `Holidays!A6:A55`.

## Master: column reference

| Col | Field | Type | Formula / rule |
|---|---|---|---|
| B | Month | In | Delivery-month bucket (`M5`, `M6`, …) |
| C | Seq | In | Sequence of the PO within its style |
| D | Style | In | Style code |
| E | PO | In | Buyer PO number |
| F | ODR. Qty | In | Order quantity |
| G | Produce Till Date | In | Pieces already produced |
| H | Balance | Calc | `=IF(OR(D="",E=""),"",IFERROR(F-G,"-"))` |
| I | Dely Date | In | Buyer delivery date |
| J | Ex-Factory (planned) | Calc | `=WORKDAY.INTL(I,-10,"0000001",Holidays)`: delivery − 10 working days |
| K | Lines | In | Sewing lines allocated |
| L | Production/LINE | In | Pieces per line per day |
| M | Days Req | Calc | `=ROUNDUP(H/(K*L),0)` |
| N | Manual Start | In | Optional override date, or `Close` to park the order |
| O | Start Date | Calc | See [scheduling rule](#scheduling-rule) |
| P | End Date | Calc | `=WORKDAY.INTL(O-1,MAX(1,ROUNDUP(H/(K*L),0)),"0000001",Holidays)` |
| Q | Projected Ex-Fac | Calc | `=WORKDAY.INTL(P,3,…)`: end + 3 working days for finishing and packing |
| R | Def (buffer) | Calc | `=NETWORKDAYS.INTL(P+1,J,…)`: working days of slack to the planned ex-factory. **Negative = late**, shown red |
| S | Fabric/Trims I/H Date | Calc | `=WORKDAY.INTL(O,-10,…)`: material must be in-house 10 working days before start |
| T | OTIF (Projected) | Calc | `=IF(Q<=I-4,1,0)`: a forecast based on the planned dates |
| U | Actual Ex-Fac | In | Typed after shipment |
| V | Shipped Qty | In | Typed after shipment |
| W | OTIF (Actual) | Calc | `=IF(OR(U="",V="",I=""),"",IF(AND(U<=I-4,V>=F),1,0))`: on time **and** in full |

Every formula is wrapped so that a blank row returns `""` and a calculation error returns `"-"`, never a raw Excel error.

## Scheduling rule

```excel
O = IF(OR($D="",$E=""), "",
       IF($N<>"", $N,                                                   ' 1. manual start wins
          IFERROR(WORKDAY.INTL(
              LOOKUP(2, 1/(($D$2:D{r-1}=$D{r})*ISNUMBER($P$2:P{r-1})),  ' 2. last END date of the same style above
                     $P$2:P{r-1}),
              1, "0000001", Holidays), "")))                            '    + 1 working day
```

- `LOOKUP(2, 1/(condition), result)` returns the **last** row above that matches the style and has a numeric end date. Each line of a style therefore picks up where the previous PO of that style finishes.
- Closed or zero-balance orders have no numeric end date, so the chain skips them.
- A typed date in **Manual Start** starts a new chain from that date.

## Conditional formatting

| Range | Rule |
|---|---|
| `O3:O200` | Text `Close` → greyed |
| `K3:L200` | Balance > 0, not closed, and lines or output blank → **amber** (the order is not being scheduled) |
| `R3:R200` | Buffer < 0 → red |

## Holidays sheet

| Area | Content |
|---|---|
| `B2` | Calendar year (input) |
| `A6:B55` | Holiday register: date and name (input). Read by every `WORKDAY.INTL` / `NETWORKDAYS.INTL` on `Master` |
| `E6:J371` | Annual working calendar built from `B2`: date, month, week number, weekday, status (`Working` / `Sunday` / `Holiday`) and whether the day is counted |
