---
name: ledger-reconciliation-matching
description: >
  Use when proposing a match between a confirmed disbursement statement line item and an existing
  open receivable LedgerEntry on the same realm, before Disbursement Reported ledger entries are
  committed and reconciliation links are created.
---

Match primarily by insured name and amount.

Only propose a candidateLedgerEntryId when it is a real, currently-open receivable returned by the
get_realm_ledger_entries tool -- never invent, guess, or reuse an id the tool did not actually
return. If the tool returns no candidates, or none of the returned candidates plausibly correspond
to a line item, there is no match: leave that line item's candidateLedgerEntryId null rather than
forcing one.
