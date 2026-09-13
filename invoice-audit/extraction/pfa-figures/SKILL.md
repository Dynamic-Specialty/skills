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
- **insuredName**: the named insured on the PFA (the borrower/party being financed), as printed.
- **agencyName**: the retail agency's name as printed, if the document names it separately from
  the finance company or insured. Leave it out if it isn't shown.

You'll also be told the invoice's own expected insured name (and agency name, if applicable).
Report two more fields:

- **insuredNameMatches**: true if the printed insured name refers to the same company as the
  expected insured name you were given.
- **agencyNameMatches**: true if the printed agency name refers to the same company as the
  expected agency name you were given. Leave it out if no agency name is shown on the document, or
  no expected agency name was given.

Getting this right matters: there have been real cases of the wrong PFA document being attached to
an invoice, and this party check is what catches that.

### What counts as a match

Judge like a person comparing two company names would, not like a text search. Treat these as the
SAME company:
- Case, punctuation, spacing, or formatting differences (e.g. "B&E TRUCKING" vs "B & E Trucking").
- A legal-entity suffix present on one side but not the other - LLC, Inc, Corp, Co, Ltd, and
  similar ("B & E Trucking" and "B & E Trucking LLC" are the same company).
- A common abbreviation or shortened form of the same name (e.g. "Intl" for "International").

Treat these as DIFFERENT companies:
- The core business name is genuinely different, even if one name happens to contain some of the
  same letters or a substring of the other. Example: "ABC" is NOT the same company as "Fabco
  Distribution", even though the letters "a-b-c" appear inside "Fabco".

When a case is genuinely ambiguous, use your best judgment about whether a reasonable person would
consider the two names the same company.

## What NOT to do

- Do not extract Total Premium, Total of Payments, the finance charge, the down payment, or the
  installment amount/count - only the financed-amount figure itself matters here.
- Do not compare the financed-amount figure against any invoice or any other document. You are not
  told what the invoice's figures say, and you have no basis to judge a figures match - that
  comparison happens entirely outside of you, deterministically. The party-name match above is the
  one judgment call you do make.
- Do not apply any notion of tolerance, rounding, or "close enough" to the financed amount.
- Do not reason about figure discrepancies or produce a figures pass/fail judgment. That
  determination is made entirely outside of you, deterministically, from the figure you extract.
- Do not respond with prose reasoning your work. Call `record_pfa_figures` once, with everything
  you found, and nothing else.
