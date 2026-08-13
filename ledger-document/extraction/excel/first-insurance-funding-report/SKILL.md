---
name: first-insurance-funding-report
description: >
  Use when parsing a PFC disbursement statement titled "Funding Report" from FIRST Insurance
  Funding. Identifiable by the "FIRST Insurance Funding" header and a "LOANS FUNDED:" / "TOTAL
  FUNDED:" footer.
---

Each loan block has a header (Loan Number, Loan Type, Insured Name, Agent Name), then a loan-level
summary row (Total Premium, Down Payment, Amount Financed, Less Retained Payment, Net Proceeds),
then one or more coverage rows (Policy Number, Funded Amount, Effective Date, Coverage, Premium,
Earned Taxes/Fees, Financed Taxes/Fees).

**Extract one line item per coverage row, never per loan.** The loan-level summary row is a
subtotal, not a line item -- its Amount Financed/Net Proceeds is the sum of that loan's coverage
rows' Funded Amounts, not a disbursement of its own. A loan can carry more than one coverage row
(e.g. separate PHYD and CRGO rows under one loan); each is its own line item, sharing the same
insured/agency/loan number from the loan header.

Per line item:
- amount: the coverage row's own **Funded Amount** -- not the loan-level Amount Financed or Net
  Proceeds, which describe the whole loan, not this one coverage.
- policyEffectiveDate: the coverage row's own Effective Date. The loan header carries no date of
  its own.
- policyNumber: take it verbatim, including a literal placeholder like "Pending" or "TBD" when
  that's what's printed -- don't invent a real-looking number.
- referenceId: the loan's Loan Number. Multiple coverage rows under the same loan legitimately
  share it -- that's not a sign of a duplicate or an error.
- insuredName / agencyName: from the loan header, the same for every coverage row under it.
- sourceFields: capture the loan header's Loan Type (e.g. "Original", "AP1", "AP2") and the
  loan-level Total Premium / Down Payment / Amount Financed / Net Proceeds figures for context,
  even though amount itself comes from the coverage row, not these.

Sanity check before finishing: the footer's "LOANS FUNDED: N" counts **loans**, not line items --
don't assume your extracted line count should equal N. What must match exactly is the dollar
total: the sum of every line item's amount should equal the footer's "TOTAL FUNDED" (the same
figure also appears once near the top, as the Payee-level "Disbursement Amount"). If your line
count came out equal to the loan count, re-check every loan block for a second coverage row you
may have missed.
