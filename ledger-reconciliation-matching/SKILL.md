---
name: ledger-reconciliation-matching
description: >
  Use when proposing a match between a ledger statement line item that has been extracted from a ledger statement
  and an existing, currently open and unmatched LedgerEntry tagged as OURS, on the same realm, before those ledger entries are
  committed and reconciliation links are created.
---

The match methodology will vary because the ledger entries created from ledger statements vary in the detail that is provided
by each submitter (a Carrier or Premium Financing Companies).

The typical match criteria uses insured name and amount, but any available information can be attempted. For example, invoice date, 
invoice number, policy effective start date, policy number, agency name. 

A proposed match needs to have a SUGGESTED matchStatus (it will be up to the user later to accept or reject the suggestion).

The proposed match also needs to provide a level of confidence in the match.

Only propose a candidateLedgerEntryId when it is a real, currently-open and unmatched OURS entry returned by the
get_realm_ledger_entries tool -- never invent, guess, or reuse an id the tool did not actually
return. If the tool returns no candidates, or none of the returned candidates plausibly correspond
to a line item, there is no match: leave that line item's candidateLedgerEntryId null rather than
forcing one.
