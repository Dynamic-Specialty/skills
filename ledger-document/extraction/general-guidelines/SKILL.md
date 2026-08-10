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
