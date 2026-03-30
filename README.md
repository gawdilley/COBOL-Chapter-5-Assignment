# RPT5000 – Year-To-Date Sales Report

**Course:** COBOL Programming – Chapter 5 Assignment
**Authors:** [Gabe Dilley](https://github.com/gawdilley) | [Garrett Finke](https://github.com/gafink01)
**Date:** March 26, 2026
**GitHub:** [COBOL-Chapter-5-Assignment](https://github.com/gawdilley/COBOL-Chapter-5-Assignment)

---

## Description

RPT5000 is a COBOL batch reporting program that reads a sequential customer master file (`CUSTMAST`) and produces a formatted Year-To-Date (YTD) Sales Report (`RPT5001`). Building on the previous chapter's report, this version adds a third level of totaling — sales rep subtotals — giving management a more granular view of performance across reps, branches, and the company as a whole.

---

## What the Program Does

### Input
The program reads fixed-length customer master records (130 characters each), each containing:
- Branch number
- Sales rep number
- Customer number
- Customer name
- Sales amount for the current year-to-date
- Sales amount for the prior year-to-date

### Processing
The program performs the following steps:

1. **Opens** the input customer master file and the output report file.
2. **Reads** each customer record using a `WITH TEST AFTER` loop, ensuring at least one read occurs before checking for end-of-file.
3. **Detects control breaks** at two levels using an `EVALUATE TRUE` structure:
   - **Sales rep break** — when the sales rep number changes, a sales rep total line is printed and the sales rep accumulators are reset.
   - **Branch break** — when the branch number changes, both a sales rep total and a branch total line are printed before processing continues.
4. **Suppresses repeated fields** on the printed customer line — when the branch or sales rep number hasn't changed, those fields are left blank to improve readability.
5. **Calculates** for each customer, each sales rep, each branch, and the grand total:
   - **Change Amount** — the difference between this year's and last year's YTD sales.
   - **Change Percent** — the percentage change relative to last year's sales. If last year's sales are zero, or if the result overflows the output field, the percentage is capped at `999.9%`.
6. **Accumulates** running totals at three levels:
   - **Sales rep totals** — reset after each sales rep break.
   - **Branch totals** — rolled up from sales rep totals and reset after each branch break.
   - **Grand totals** — accumulated from branch totals across the entire run.
7. **Manages pagination** — when the line count exceeds 55 lines per page, a new page begins automatically with fresh heading lines.
8. **Controls line spacing** using a `SPACE-CONTROL` field passed into a shared write routine, allowing flexible single or double spacing throughout the report.

### Output
The printed report (`RPT5001`) includes:

- **Report headings** on each page with the current date, time, report title, report ID, and page number.
- **Column headers** for Branch #, Sales Rep #, Customer #, Customer Name, Sales This YTD, Sales Last YTD, Change Amount, and Change Percent.
- **One detail line per customer**, with branch and sales rep numbers suppressed when unchanged from the previous line.
- **Sales rep total lines** (marked with `*`) after the last customer for each sales rep.
- **Branch total lines** (marked with `**`) after the last sales rep in each branch.
- **A grand total line** (marked with `***`) at the end of the report.

---

## Example Output

```
DATE:  03/26/2026           YEAR-TO-DATE SALES REPORT  PAGE:    1
TIME:  14:22                                                         RPT5000

BRANCH  SLSREP  CUST    CUSTOMER NAME                 SALES           SALES           CHANGE         CHANGE
 NUM     NUM    NUM                                   THIS YTD        LAST YTD        AMOUNT         PERCENT
------  ------  ------  --------------------   -------------   -------------   -------------   ------
 01       01    00101   ACME CORPORATION              15,000.00       12,500.00       2,500.00-       20.0
                        SMITH HARDWARE                 8,200.00        9,100.00         900.00-        9.9-
                                SALESREP TOTAL        23,200.00       21,600.00       1,600.00        7.4 *

         02     00201   JONES ELECTRIC                11,500.00       11,500.00            .00          .0
                        METRO SUPPLY CO                4,750.00        3,200.00       1,550.00        48.4
                                SALESREP TOTAL        16,250.00       14,700.00       1,550.00        10.5 *

                              BRANCH TOTAL            39,450.00       36,300.00       3,150.00         8.7 **

 02       01    00301   LAKE CITY TOOLS               22,000.00       19,800.00       2,200.00        11.1
                        RIVERSIDE MFG                  9,100.00        9,100.00            .00          .0
                                SALESREP TOTAL        31,100.00       28,900.00       2,200.00         7.6 *

                              BRANCH TOTAL            31,100.00       28,900.00       2,200.00         7.6 **

                           GRAND TOTAL                70,550.00       65,200.00       5,350.00         8.2 ***
```

---

## New Concepts Used

- **Three-level control break processing** — detecting breaks at both the sales rep and branch level using a single `EVALUATE TRUE` block, with each level triggering the appropriate subtotal routines before processing continues
- **`EVALUATE TRUE` for control breaks** — using condition-based evaluation instead of nested `IF` statements to cleanly handle multiple break scenarios (EOF, first record, branch break, sales rep break, and no break)
- **Level-88 condition names** — defining `CUSTMAST-EOF` and `FIRST-RECORD` as named conditions on their parent switches, allowing readable `SET CUSTMAST-EOF TO TRUE` and `IF FIRST-RECORD` usage throughout the procedure division
- **`WITH TEST AFTER` on `PERFORM UNTIL`** — guaranteeing the loop body executes at least once before the EOF condition is tested, which is necessary when the first read happens inside the loop
- **`FIRST-RECORD` switch logic** — using a dedicated first-record flag to handle initialization of control fields and suppress repeated branch/sales rep numbers on the first detail line without special-casing the read logic
- **Field suppression on detail lines** — blanking out the branch and sales rep number fields in the customer line when those values haven't changed, improving visual readability of the report
- **`ON SIZE ERROR` on `COMPUTE`** — trapping arithmetic overflow on the percentage calculation and substituting `999.9` rather than allowing a runtime error or garbled output
- **`SPACE-CONTROL` variable line spacing** — using a single shared write paragraph (`350-WRITE-REPORT-LINE`) with a `SPACE-CONTROL` field to handle both single and double spacing, avoiding duplicate write logic
- **`RECORDING MODE`, `LABEL RECORDS`, `RECORD CONTAINS`, `BLOCK CONTAINS`** — explicitly declaring file attributes in the `FD` entries for both the input and output files
- **Asterisk level indicators on total lines** — appending `*`, `**`, and `***` to sales rep, branch, and grand total lines respectively as a standard reporting convention to visually distinguish totaling levels

---

## Authors

| Name | Profile |
|------|---------|
| Gabe Dilley | [GitHub](https://github.com/gawdilley) |
| Garrett Finke | [GitHub](https://github.com/gafink01) |
