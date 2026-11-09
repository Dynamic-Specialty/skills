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

## Layout

- One block per policy: its transaction rows, then a "POLICY TOTAL" row. The last page ends with a
  "STATEMENT TOTAL" row. Never return either total row as a line.
- The policy column reads like "IMP F431009 00-00". Return it as printed in `policyNumber`.
- A row at the top of a page with no policy or insured belongs to the policy at the end of the
  previous page.
- Each printed row is its own transaction, even when one policy has several in the same month.
- `carrierName` is Great American Insurance. Leave `agencyName` empty.

Printed columns: trans type, trans date (month and year), gross premium, comm %, net premium, past
due pay on receipt, current due, future due, comments. In the page text, the last three amounts of
a row are past due, current due and future due.

## Lines from a row

Return every transaction row, whether or not it is payable now. A row gives:

- A premium line: `amount` is the gross premium. `itemType` is `PREMIUM`, or `PREMIUM_RETURN` when
  the gross premium is negative.
- A commission line, only when comm % is above 0: `amount` is the gross premium times comm %.
  `itemType` is `CARRIER_COMMISSION`, or `CARRIER_COMMISSION_RETURN` when the gross premium is
  negative.

A row with 0% commission gets a premium line only. This statement has no fees, so never return a
fee line. Give amounts without a sign: the `itemType` carries the direction.

On every line:

- `reasonCode`: New Business is `BIND`, Endorsement is `ENDORSEMENT`, Cancellation is `CANCELLED`.
- `policyEffectiveDate`: the "pol eff date" of the policy.
- `sourceFields`: `transType`, `transDate`, `commPercent`, `netPremium`, the `comments` text (for
  example "Premium Mismatch"), and the three due amounts as printed: `pastDue`, `currentDue`,
  `futureDue`.

## Payable now

- If `pastDue` or `currentDue` is not 0.00, set `dueOnThisStatement` to `true` on every line of the
  row. `dueDate` is the date in the "current due" heading.
- If both are 0.00, set `dueOnThisStatement` to `false` on every line of the row. Such lines are
  discarded afterwards. Nothing else decides this.
- A "Payment" row is the exception. Its three due amounts show 0.00 because the payment is already
  inside the current due of its policy. Always return it: one line with no `itemType`, the printed
  amount with its printed sign, `dueDate` from the "current due" heading, and `dueOnThisStatement`
  empty. Put `Payment` in `sourceFields`.
- A row whose due amounts do not add up to its net premium is still returned. Say so in
  `sourceFields.comments`.

Examples from the page text (gross, comm %, net, past due, current due, future due):

- `$ 696.00 20.00 $ 556.80 $ 0.00 $ 556.80 $ 0.00` is payable now.
- `$ (1,553.00) 20.00 $ (1,242.40) $ 0.00 $ 0.00 $ (1,242.40)` has only a future due. Return its
  lines with `dueOnThisStatement` `false`.
- `Payment 08/13/2026 $ (534.20) 0.00 $ (534.20) $ 0.00 $ 0.00 $ 0.00 Premium Mismatch` is a Payment
  row. Return it with amount -534.20.

## Statement-level figures

- `documentDate`: the "date mailed".
- `dueDate`: the date in the "current due" column heading.
- `declaredTotal`: the "current due" figure on the "STATEMENT TOTAL" row.
- `declaredTotalNotDue`: leave empty.
