# Review findings and fixes

Both workbooks were reviewed before publication. Every fix was applied to the working files first, then carried into the anonymised samples.

**Verification approach:** each workbook was recalculated in a second engine (LibreOffice) and every formula result was compared with the original. A fix only counts when the plan's numbers are unchanged, unless the change was the intended one.

## Order planner

### 1. Start Date formula had been overwritten

- **Found:** the chaining formula in `Start Date` existed only in rows 3–30. In 163 of the 191 order rows it had been typed over with dates or `Close`, so new orders no longer picked up from the previous order of the same style.
- **Fix:** the formula was restored in every row (3–200). Each typed value that differed from what the formula would produce (151 rows: 118 dates, 33 `Close`) was moved into **Manual Start**, where the formula treats it as an override.
- **Check:** a Python re-implementation of the `WORKDAY.INTL` chaining reproduced Excel's results on the untouched rows exactly. After the fix, every Start, End, Ex-Factory, buffer and OTIF value is identical to before.

### 2. OTIF measured a forecast

- **Found:** the column headed *Actual EX-Fac* was a formula (end date + 3 working days), and OTIF only compared that projected date with delivery. "In full" was never tested.
- **Fix:** renamed to *Projected Ex-Fac* and *OTIF (Projected)*. Added the inputs **Actual Ex-Fac** and **Shipped Qty**, and **OTIF (Actual)** = on time (≥ 4 days before delivery) **and** shipped qty ≥ order qty.

### 3. Unscheduled open orders

- **Found:** 18 orders with a balance had no lines or output per line, so they never got an end date.
- **Fix:** an amber rule highlights them. The rows themselves need a planning decision.

### 4. Fragile "Close" highlight

- **Found:** the rule compared each cell with whatever was in `O8`.
- **Fix:** the rule now tests for the text `Close` over `O3:O200`.

### Not changed

- **Holiday names:** the holiday register has dates but no names. These are left for the planner to fill in.

## Machine capacity planner

### 5. GUIDE pointed at old cells

The GUIDE sent users to `DJ2`, column `DN` and column `DI`. The real cells are `HA2` (pricing check), `HE5:HE47` (validation list) and `GY36:GY61` (thermal variants). The text is corrected.

### 6. Month views were hard-coded to Aug–Nov 2026

- **Found:** the month list in `DEMAND BY DATE!BD27:BD30` held the typed numbers 8–11, combined with the start year. Moving the plan start (which the GUIDE invites) broke **502 formulas** across `MD VIEW`, `DEMAND BY DATE`, `MC MAP` and `WORKINGS`. A plan crossing into January would also have counted the blank days beyond the horizon as January days.
- **Fix:** the month numbers, labels and first-row lookups are derived from the plan start with `EDATE`. Day counts now ignore zero-dated rows.
- **Check:** with the current start (1-Aug-2026) every value is unchanged. With the start moved to 10-Nov-2026 the months read *Nov 2026 – Feb 2027* and the errors fall from 502 to none, once a new as-of date is picked.

### 7. Dashboard pickers had no dropdowns

The cell note on `DASHBOARD!D4` says "use the dropdown arrow", but `C4`, `D4` and `E4` had no validation. Lists were added:

- **C4:** `DAY` / `MONTH END` / `MONTH PEAK`.
- **D4:** plan dates, sized by the horizon in `LINE PLAN!B3`.
- **E4:** months.

### 8. Bottleneck header showed the wrong number

`"TOP 6 BOTTLENECKS OF "&$H$6&…` read the utilisation cell, giving *"TOP 6 BOTTLENECKS OF 0.878260869565217 SHORT GROUPS"*. It now reads the short-group count `$J$6`, giving *"TOP 6 BOTTLENECKS OF 5 SHORT GROUPS"*. This is the only value that changed.

### Not changed

- **Plan coverage:** the plan grid is filled to 23-Oct while the horizon runs to 13-Nov, so the last 21 days carry no demand. That is a planning gap, not a model fault.
- **Archives:** `PPC` and `MACHINE MASTER` are already labelled *SUPERSEDED* and hidden.
- **Engine quirk:** for `THERMAL` / `VACCANT` rows, `MC MAP!I16:I17` shows `ALIAS NOT FOUND` in Excel, where *No embl. data* is meant. This comes from how Excel's `INDEX` treats a blank cell, and it doesn't affect any number.

## Style crosswalk

### 9. Style codes are written differently in the two modules

The two modules can't be linked automatically yet. Matched after removing spaces and hyphens (anonymised codes):

| Order planner style | Capacity planner style / recipe | Note |
|---|---|---|
| 4110, 4115, 4117, 4300, 4301, 5501, 5520, 6108, 6109, 6135, 6137, 6820, 7926 | same code | direct |
| RS17 / RS27 | RS-17 / RS-27 | hyphen only |
| TH01, TH02, TH04, TH05, TH06 | TH-01 … TH-06 | hyphen only |
| RS26 | RS 26/27 → recipe RS-27 | combined board name |
| 4406, 4406P, 4410P | 4406/10 → recipe *Panty* | combined board name |
| **TH07** | **no recipe** | ordered style with no machine recipe. If planned, it would cost zero machines |

**Recommended next step:** a single `STYLE MASTER` table (code, board name, recipe column, garment type) that both modules look up, replacing the alias table and free-typed codes.

## Anonymisation checks

- **Recalculated:** both samples were recalculated from scratch.
- **Capacity planner:** every formula result is identical to the fixed working file, or differs only by the masked style name (1,441 numeric results that are style codes).
- **Order planner:** every date and days-required value is identical.
- **Leak scan:** run over every part of both files (sheets, shared strings, comments, charts, headers and footers, document properties) for the company name, PO numbers, original style codes, machine brands and models, and author names. Nothing was found.
