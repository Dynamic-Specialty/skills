---
name: invoice-audit-signed-proposal-extraction
description: >
  Use when extracting category-tagged figures, the named insured/agency, and signature status
  from a signed insurance proposal document, as part of auditing an invoice before booking.
---

# Signed proposal figure extraction

## Identifying the document

A signed insurance proposal shows the insured's coverage, pricing breakdown, and a signature or
signed-date confirmation. It is not a policy declarations page, an invoice, or a blank template. If
the attached document doesn't look like a signed proposal, extract what figures you can find and
proceed anyway - do not refuse to answer.

## What to extract

For every distinct dollar figure printed on the document, extract three things:

- **label**: the text exactly as printed next to the figure (e.g. "State Tax", "Policy Fee",
  "Total Premium"). Do not paraphrase or normalize it.
- **category**: exactly one of `Premium`, `Tax`, `Fee`, `RevenueFee`, `Commission`, `Financing`,
  `Total`.
- **amount**: the dollar amount as printed.

Category meanings:
- **Premium**: the insurance premium itself, and any premium return/credit.
- **Tax**: any tax charged on the policy (state tax, surplus lines tax, stamping fee).
- **Fee**: any fee that isn't specifically called out as a revenue/placement fee (policy fee,
  carrier fee, admin fee, filing fee).
- **RevenueFee**: use this only when the document itself separately labels a fee as the agency's
  own revenue/placement fee, distinct from other fees. If the document doesn't call this out
  separately, use `Fee` instead - do not guess that an unlabeled fee is a revenue fee.
- **Commission**: agency or carrier commission, if shown (most signed proposals do not show this to
  the insured - if you don't see it, don't invent it).
- **Financing**: any amount related to premium financing (due from a finance company, financed
  amount, down payment due).
- **Total**: the document's own printed grand total. Exactly one figure must carry this tag.

## Extracting the insured, agency, and signature status

Alongside the figures, also pass these three to `record_extracted_figures`:

- **insuredName**: the *named insured* on the proposal. This document has a section titled
  **"Insured Details"** containing two separate fields: **"Company"** and **"Primary Contact"**.
  Always read `insuredName` from the **"Company"** field. **Never** read it from
  **"Primary Contact"** - that field is the individual who is expected to sign the document on the
  company's behalf, not the insured itself, and its name will often differ from the company name
  (e.g. Company: "B & E Trucking", Primary Contact: "Bobby Earl Smith" - in that case `insuredName`
  is "B & E Trucking").
- **agencyName**: the retail agency's name as printed, if the document names it separately from the
  carrier or insured. Leave it out if it isn't shown.
- **signedAndDated**: true only if the document shows an actual completed signature and date - not
  a blank signature line, not a template placeholder, not just a printed name with no accompanying
  signature mark or date next to it.

You'll also be told the invoice's own expected insured name (and agency name, if applicable).
Report two more fields alongside the above:

- **insuredNameMatches**: true if the printed insured name refers to the same company as the
  expected insured name you were given.
- **agencyNameMatches**: true if the printed agency name refers to the same company as the
  expected agency name you were given. Leave it out if no agency name is shown on the document, or
  no expected agency name was given.

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

## Avoiding double-counting

If the document shows both a subtotal (e.g. "Total Fees: $150") and itemizes each fee separately
underneath it, extract the itemized components, not the subtotal. Extracting both would double the
category's true total.

## The Total figure - critical

Exactly one extracted figure must be tagged `Total`, and its amount must be a literal number
printed on the document itself - the document's own stated grand total. Never compute this
yourself by summing the other figures you extracted. A downstream check compares your `Total`
against the sum of your other figures specifically to catch a missed or double-counted line - if
you derive `Total` from your own sum instead, that check becomes meaningless.

If the document has no clear single grand total, use your best judgment about which printed figure
represents the overall amount, and tag that one `Total`.

## What NOT to do

- Do not read `insuredName` from the "Primary Contact" field in Insured Details - that is the
  signer, not the insured. Read it from the "Company" field.
- Do not compare the dollar figures against any invoice or any other document. You are not told
  what the invoice's figures say, and you have no basis to judge a figures match - that comparison
  happens entirely outside of you, deterministically. The party-name match above is the one
  judgment call you do make.
- Do not apply any notion of tolerance, rounding, or "close enough" to the figures.
- Do not reason about figure discrepancies or produce a figures pass/fail judgment. That
  determination is made entirely outside of you, deterministically, from the figures you extract.
- Do not respond with prose reasoning your work. Call `record_extracted_figures` once, with every
  figure you found, and nothing else.
