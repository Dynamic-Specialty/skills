---
name: general-guidelines
description: >
  General extraction guidance that applies regardless of vendor format -- not tied to recognizing
  any one specific report layout or title. Use alongside whatever vendor-specific skill (if any)
  also matches the document.
---

## Combining split premium/taxes rows

Some statements report one policy's premium and its taxes/fees as two separate rows under the
same account/loan reference, rather than one combined row. When you see this pattern -- two rows
sharing the same account number or loan reference, one of them labeled something like
"TAXES/FEES" (usually with placeholder/zero dates) and the other carrying the real policy number
and effective dates -- treat them as one combined disbursement, not two separate line items.

The two rows can appear in **either order** -- sometimes the taxes/fees row first, sometimes the
policy row first. Order carries no meaning here; always combine both into a single line item
regardless of which comes first.

For the combined line item:
- amount: the sum of both rows' own amounts (often already printed as a subtotal directly below
  them) -- not a broader loan/financed-amount figure that may appear elsewhere in the document
  (e.g. in a header row for the whole account), which can be a different, larger number than what
  was actually disbursed in this combined line.
- policyNumber and policyEffectiveDate: take these from whichever row carries the real policy
  number and real dates -- ignore the taxes/fees row's dates, which are typically placeholders
  (e.g. 00/00/0000) rather than real values.

## Carrier statements

Applies only to a statement from an insurance carrier (agency bill or commission statement), not
to a premium finance company statement. For a finance company statement leave `itemType`,
`reasonCode`, `dueDate` and `dueOnThisStatement` empty and extract as described above.

A carrier prints the parts of one transaction together, often in one row. Reconciliation needs
them apart. Return one line item per part, not one per printed row:

- `itemType`: one of `PREMIUM`, `PREMIUM_RETURN` (a negative premium), `CARRIER_COMMISSION`,
  `CARRIER_COMMISSION_RETURN`, `CARRIER_PAIDFEE`, `CARRIER_PAIDFEE_RETURN`. Use these names only.
- `reasonCode`: one of `BIND`, `RENEWAL`, `ENDORSEMENT`, `CANCELLED`, `REINSTATEMENT`, from the
  carrier's transaction type. The vendor-specific skill gives the mapping. Leave it empty if there
  is none.
- `amount`: signed. The vendor-specific skill says how each part is signed.
- `dueDate`: the date of the statement column the line was printed under, as ISO-8601.
- `dueOnThisStatement`: false when that column is for a later statement (for example a "future
  due" column), true otherwise.

Extract every line, including lines that are due later. Never skip one. Subtotal and total rows
are not line items: do not return them, use them to check your own extraction.

Statement-level figures for a carrier statement:

- `dueDate`: the payment due date printed for the amount payable now.
- `declaredTotal`: the printed total of the amount payable now (not a total of every column).
- `declaredTotalNotDue`: the printed total of the lines due later, if the statement prints one.

As above, these are printed numbers. Never add them up yourself.

## A part of a larger document

When the request carries a line like "Part data: this file holds pages 6-9 of 14 of the
statement", the file is only part of the statement. Extract line items only from those pages.

If the line also says the first page repeats the previous part's last page, that first page is
context only. Do not return its line items. Use it only to learn the policy and insured of a
continuation row at the top of the first real page, which has none of its own.

Return statement-level figures (`documentDate`, `declaredTotal`, `dueDate` and so on) only if they
are printed on the pages you were given. Otherwise leave them empty.
