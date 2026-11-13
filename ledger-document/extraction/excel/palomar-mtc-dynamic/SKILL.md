---
name: palomar-mtc-dynamic
description: >
  Use when parsing a carrier statement workbook from Palomar for Motor Truck Cargo, named like
  "Motor Truck Cargo Palomar Statement - Dynamic Specialty (PSIC-12034) - October 2026". One tab,
  "MTC Dynamic". Identifiable by the columns "Agency Name", "GL Line of Business", "Policy Number",
  "Insured Name", "Effective Date", "Due Date", "Bill Date", "Gross Premium", "Agency Commission",
  "Policy Balance" and "Agency Code".
---

This is a carrier statement. The general guidelines for carrier statements apply. This skill says
how this layout maps onto them.

The rules below are applied by code from `mapping.json` in this folder. Keep the two in step.

## Layout

- One tab. The header is row 1.
- One row per policy transaction. There are no policy blocks and no subtotals.
- The data ends at the first blank row. Below it are the beneficiary and bank details. Never
  return those as lines.
- Each printed row is its own transaction, even when one policy has several rows (one per due date).
- `insuredName`: "Insured Name". `policyNumber`: "Policy Number", as printed.
- Leave `agencyName` and `carrierName` empty. "Agency Name" is Dynamic itself.
- Dates in the sheet are real dates. "Due Date" and "Bill Date" are not used.

## Lines from a row

| Column | `itemType` | Return type |
| --- | --- | --- |
| Gross Premium | `PREMIUM` | `PREMIUM_RETURN`, when the column is negative |
| Agency Commission | `CARRIER_COMMISSION` | `CARRIER_COMMISSION_RETURN`, when the column is positive |

- The commission is printed with the opposite sign of the premium. A negative commission on a
  positive premium is a normal `CARRIER_COMMISSION`. The code reverses the sign for this column.
- `amount` is the printed figure without its sign.
- "Policy Balance" is Gross Premium plus Agency Commission. Never return it as a line.

On every line:

- `reasonCode`: leave empty. The sheet has no transaction type. Do not guess it from the sign.
- `policyEffectiveDate`: the "Effective Date" of the row.

## Statement-level figures

- `declaredTotal`: the "Policy Balance" figure on the row that starts with "Beneficiary Name". It
  is the sum of the "Policy Balance" column.
- `documentDate`: leave empty. The sheet prints no statement date.
- `dueDate`: leave empty. `dueOnThisStatement`: leave empty on every line.
- `declaredTotalNotDue`: leave empty.
