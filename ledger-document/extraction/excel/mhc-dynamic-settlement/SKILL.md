---
name: mhc-dynamic-settlement
description: >
  Use when parsing a carrier settlement workbook from MHC with one tab per month, named like
  "Dynamic Settlement 9.2026". Identifiable by the columns "Insured Name", "Policy No.", "Date
  Bound", "Pol Eff Date", "Trans Eff Date", "Transaction Type", "Premium", "Capital Contribution",
  "MembershipFee", "State Tax", "Admin Fee", "SMFI Fee", "ReinstateFee", "Veh Change", "Total",
  "Commission" and "Balance Due".
---

This is a carrier statement. The general guidelines for carrier statements apply. This skill says
how this layout maps onto them.

**Provisional:** the fee and tax split below, and the treatment of "Reinstatement Notice" rows, are
awaiting confirmation from accounting.

The rules below are applied by code from `mapping.json` in this folder. Keep the two in step.

## Which tab

- The workbook holds one tab per month, newest first. The statement is the first tab only.
  Ignore every other tab, even if it looks complete.
- Tab names are inconsistent (spaces, suffixes such as "AL"). Go by the order, not the name.

## Layout

- One row per transaction. There are no policy blocks and no subtotals.
- The last row has no insured and no policy. It holds the totals of "Total", "Commission" and
  "Balance Due". Never return it as a line.
- Each printed row is its own transaction, even when one policy has several rows.
- `insuredName` and `policyNumber`: as printed. Leave `agencyName` empty. Leave `carrierName`
  empty unless the sheet prints it.
- "Veh Change" is a count of vehicles, not money. Never make a line from it.

## Lines from a row

A row gives one line for each of these columns that is not 0:

| Column | `itemType` | Return type, when the column is negative |
| --- | --- | --- |
| Premium | `PREMIUM` | `PREMIUM_RETURN` |
| Commission | `CARRIER_COMMISSION` | `CARRIER_COMMISSION_RETURN` |
| Capital Contribution | `CARRIER_FEE` | `CARRIER_FEE_RETURN` |
| Admin Fee | `CARRIER_FEE` | `CARRIER_FEE_RETURN` |
| SMFI Fee | `CARRIER_FEE` | `CARRIER_FEE_RETURN` |
| State Tax | `CARRIER_TAX` | `CARRIER_TAX_RETURN` |
| MembershipFee | `CARRIER_PAIDFEE` | `CARRIER_PAIDFEE_RETURN` |
| ReinstateFee | `CARRIER_PAIDFEE` | `CARRIER_PAIDFEE_RETURN` |

- Each column is its own line. Never combine columns.
- `amount` is the printed figure without its sign. The sign of that column decides the type.
- A cancellation or a negative endorsement shows negative figures, so its lines are return types.
- Never return "Total" or "Balance Due" as a line. They are sums of the lines.

On every line:

- `reasonCode`, from "Transaction Type":
  - New Business is `BIND`.
  - Renewal is `RENEWAL`.
  - Endorsement/Change is `ENDORSEMENT`.
  - Cancellation Endorsement is `CANCELLED`.
  - Reinstatement Endorsement and Reinstatement Notice are `REINSTATEMENT`.
- `policyEffectiveDate`: the "Pol Eff Date" of the row.
- `sourceFields`: only `transactionType` and `transEffDate`. Add nothing else. A long workbook
  has about 150 lines, and a longer answer is cut off before it ends.

A "Reinstatement Notice" row has only a ReinstateFee. It gives one `CARRIER_PAIDFEE` line and no
commission. Its Trans Eff Date can be in the future. Return it anyway.

## Statement-level figures

- `declaredTotal`: the "Balance Due" figure on the last row (the one with no insured). It is
  the Total less the Commission.
- `documentDate`: leave empty. The sheet prints no statement date.
- `dueDate`: leave empty. `dueOnThisStatement`: leave empty on every line.
- `declaredTotalNotDue`: leave empty.
