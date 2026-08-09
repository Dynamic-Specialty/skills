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

## Tool usage

Use the get_realm_ledger_entries tool to fetch real candidates for this realm: openOnly=true,
direction=OURS. Do not restrict by entry type -- depending on the realm, an OURS entry can be a
RECEIVABLE or a PAYABLE; entry_type does not determine this, direction does. Do not use any other
tool here -- creating entries and reconciliation links happens separately, after matches are
proposed.

## Output contract

For each confirmed line item, produce:
- candidateLedgerEntryId: the matched entry's real id, as returned by get_realm_ledger_entries, or
  null if no reasonable candidate exists (see above -- never invent, guess, or reuse an id the
  tool did not actually return)
- confidenceScore: an integer 0-100
- matchReason: a brief explanation of the match, or of why none was found

You do not set matchStatus yourself -- every proposal you produce is recorded as SUGGESTED
automatically once you provide the fields above; a human accepts or rejects it later.
