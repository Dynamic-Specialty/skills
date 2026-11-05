---
name: great-american-agency-bill-statement
description: >
  Use when parsing a carrier statement titled "AGENCY BILL STATEMENT" from Great American
  Insurance. Identifiable by the "GREAT AMERICAN INSURANCE" mail-payment address, "STATEMENT
  MONTH", "MASTER PRODUCER", and table columns "gross premium", "comm %", "net premium", "past due
  pay on receipt", "current due" and "future due".
---

This is a carrier statement. The general guidelines for carrier statements apply. This skill says
how this layout maps onto them.

**Provisional:** the commission sign rule and the treatment of a "Payment" row below are awaiting
confirmation from accounting.

## Layout

- One block per policy. The policy column reads like "IMP F431009 00-00". Extract it as printed
  into `policyNumber`. Which part of it is the matching key is not decided yet.
- Each block has one or more transaction rows, then a "POLICY TOTAL" row. The last page ends with
  a "STATEMENT TOTAL" row.
- A policy can run across a page break. A row at the top of a page with no policy or insured
  belongs to the policy at the end of the previous page.
- The same policy can have several transactions in one statement, for example two endorsements in
  the same month. Each printed row is its own transaction.
- Do not return "POLICY TOTAL" or "STATEMENT TOTAL" rows as line items.
- The carrier is Great American Insurance: set `carrierName` to that. Leave `agencyName` empty.

## One printed row becomes up to two line items

Printed columns: trans type, trans date (month and year only), gross premium, comm %, net premium,
past due, current due, future due, comments.

- Premium line: `amount` is the gross premium as printed. `itemType` is `PREMIUM`, or
  `PREMIUM_RETURN` when the gross premium is negative.
- Commission line, only when comm % is above 0: `amount` is net premium minus gross premium, taken
  from the printed numbers (normally a negative number), so that the two lines add up to the net
  premium. `itemType` is `CARRIER_COMMISSION`, or `CARRIER_COMMISSION_RETURN` when the gross
  premium is negative.
- A row with 0% commission is premium on which no commission is paid. It gets a premium line only.
  This statement shows no fee on its own, so never return a fee line.
- `reasonCode`: New Business is `BIND`, Endorsement is `ENDORSEMENT`, Cancellation is `CANCELLED`.
  A "Payment" row has no reason code: return one premium-kind line with `itemType` empty, the
  printed amount, and `Payment` in `sourceFields`.
- Put `transType`, `transDate`, `commPercent`, `netPremium` and any `comments` text (for example
  "Premium Mismatch" or "Late Entered") into `sourceFields`.
- `policyEffectiveDate` is the "pol eff date" of the policy.

## Which column a line belongs to

- If the row has an amount under "current due" (or "past due pay on receipt"), both of its lines
  have `dueOnThisStatement` true and `dueDate` is the date in the "current due" heading.
- If the row has an amount under "future due", both of its lines have `dueOnThisStatement` false
  and `dueDate` is the date in the "future due" heading.
- A row is normally in only one of the two columns. If the columns do not add up to the net
  premium, extract the row as printed and say so in `sourceFields.comments`.

## Statement-level figures

- `documentDate`: the "date mailed".
- `dueDate`: the date in the "current due" column heading.
- `declaredTotal`: the "current due" figure on the "STATEMENT TOTAL" row.
- `declaredTotalNotDue`: the "future due" figure on the "STATEMENT TOTAL" row.
- The `amount` of all lines with `dueOnThisStatement` true should add up to `declaredTotal`.
