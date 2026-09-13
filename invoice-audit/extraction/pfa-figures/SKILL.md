---
name: invoice-audit-pfa-figures-extraction
description: >
  Use when extracting the financed amount and signature status from a signed premium finance
  agreement (PFA/finance agreement) document, as part of auditing a financed invoice before
  booking.
---

# PFA figure extraction

## Identifying the document

A signed premium finance agreement (PFA) is issued either by a third-party premium finance company
or by Dynamic's own in-house financing program. It shows a payment schedule and financing terms
for the insured's premium - it is not the insurance proposal itself, a policy declarations page, or
an invoice. If the attached document doesn't look like a PFA, extract what you can find and proceed
anyway - do not refuse to answer.

## What to extract

Only two things matter for this audit:

- **financedAmount**: the amount actually financed by the premium finance company. Look for a
  field labeled something like "Amount Financed", "Total Financed", "Principal Balance", or
  "Amount to be Financed" - the balance being paid off in installments, after any down payment.
  This is **not** the same as:
  - **Total Premium** / **Cash Price** - the full premium before any down payment is subtracted.
  - **Total of Payments** - the amount financed *plus* the finance charge (interest) that
    accumulates over the installment schedule.
  If the document only prints "Total of Payments" and a separate "Finance Charge" but never prints
  "Amount Financed" directly, do not compute it yourself by subtracting - look for the actual
  printed amount-financed figure first. Only if no such figure is printed anywhere on the document,
  use your best judgment about which printed figure most closely represents the principal amount
  being financed.
- **signedAndDated**: true only if the document shows an actual completed signature and date - not
  a blank signature line, not a template placeholder, not just a printed name with no accompanying
  signature mark or date next to it.

## What NOT to do

- Do not extract Total Premium, Total of Payments, the finance charge, the down payment, or the
  installment amount/count - only the financed-amount figure itself matters here.
- Do not compare this figure against any invoice or any other document. You are not told what the
  invoice says, and you have no basis to judge a match.
- Do not apply any notion of tolerance, rounding, or "close enough."
- Do not reason about discrepancies or produce a pass/fail judgment. That determination is made
  entirely outside of you, deterministically, from the figure you extract.
- Do not respond with prose reasoning your work. Call `record_pfa_figures` once, with the financed
  amount and signature status, and nothing else.
