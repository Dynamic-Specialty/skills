---
name: spreadsheet-mapping
description: >
  Use for every statement that is a spreadsheet (xlsx, xls, csv). You do not write the statement lines.
  You read the sheets with tools and record rules for each table. Code then applies the rules to every row.
---

You have tools to read the file and to record rules. The lines are made by code, row by row, from your rules.
Never write lines in your answer.

## Process

1. Call `list_sheets`, then `describe_sheet` for each sheet the carrier skill says counts.
2. Find the tables. A sheet can hold one table, or many stacked ones, each with its own title, header, rows and total.
   A header can repeat inside one table.
3. Read the title and header rows with `read_rows`. Read a few data rows and the rows around any total.
4. For each table, call `record_table`. Read the answer. If it does not tie to the printed total, find out why and call it again.
5. Call `record_declared_total` for the statement's own grand total, when the sheet prints one.
6. When every table the carrier skill names is recorded and ties, stop. Say in one line what you recorded.

## What a table is

A table is a run of data rows read with one set of rules. Choose `firstRow` and `lastRow` so that the title, the header, the total
and the blank rows are outside it. If a table has a subtotal or a repeated header inside it, keep the table whole and skip those rows.

## The rules of a table

- `name`: the table's title, or a short description.
- `sheet`, `firstRow`, `lastRow`: where its data rows are.
- `skipRowsWhen`: a row is skipped when any of these holds. Use it for subtotals, repeated headers and footers.
  Each has a `column`, an `operator` (`EMPTY`, `NOT_EMPTY`, `EQUALS`, `STARTS_WITH`, `CONTAINS`) and a `value`.
  Fully blank rows are always skipped.
- `fields`: where the identity of a row comes from. Each of `insuredName`, `agencyName`, `carrierName`, `policyNumber`,
  `policyEffectiveDate` and `referenceId` is a `column` letter, or a fixed `value`. Leave out what the sheet does not print.
- `carryDown`: the field names that are printed only on the first row of a group and must repeat on the rows below it.
- `lines`: one rule for each kind of line a row can give. See below.
- `expectedTotal`: the cell where the sheet prints this table's own total. Name it whenever there is one.

## Line rules

A row gives one line for each rule whose amount column is not empty and not 0.

- `amountColumn`: the column letter of the figure.
- `itemType`: the type when the figure is positive. `returnItemType`: the type when it is negative.
  The line's `amount` is always the figure without its sign.
- `flipSign`: true when the sheet prints this column with the opposite sign, so that a negative figure means a charge.
- `typeFrom`: use this instead of a fixed type when the type comes from the words in a column of the same row.
  It has a `column` and `byValue`, a map from the sheet's own words to `itemType` and `returnItemType`.
  A row whose words are not in the map is reported, not guessed.
- `reason`: the `constant` reason code, or a `column` and a `byValue` map from the sheet's words to reason codes.
- `policyColumn`: when each money column has its own policy column. An empty cell or 0 falls back to the row's `policyNumber`.

Use the type and reason names that the carrier skill and the general guidelines give. Names that do not exist are rejected.

## Rules of thumb

- Column letters, rows and sheets are as `read_rows` shows them. Rows and sheets count from 1.
- Check one data row by hand against your rules before you record. Then let the tie check confirm it.
- A table that ties by a few cents is fine. A table that is off by a whole figure is wrong. Look for a column you left out or counted twice.
- Do not make a line from a total, a subtotal or a count. Do not make a line from a column that is the sum of other columns you already use.
- If a table cannot be expressed in these rules, record the part that can be and say in your final line what is left out and why.
