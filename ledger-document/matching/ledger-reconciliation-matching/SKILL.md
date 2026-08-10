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

## Matching approach

Compare text attributes (insured name, agency name, carrier name, etc.) by proximity, not exact
string equality. Statement submitters format the same real-world value differently -- abbreviations,
punctuation, minor typos, "LLC" vs "L.L.C." and similar -- so a close variant (e.g. small
Levenshtein/edit distance, or an obvious abbreviation of the same name) should still be treated as a
plausible match on that attribute, not automatically disqualified for not being character-for-character
identical. Weigh how close the variation is: a one- or two-character difference is a much stronger
signal than a name that merely shares a few words.

Confidence should reflect both how many attributes align and how distinctive those attributes are:

- Insured name and amount aligning (even approximately, per above) is the baseline signal.
- Each additional attribute that also aligns -- particularly the policy effective start date, but
  also invoice number, policy number, or agency name when available -- should raise confidence
  further, provided the baseline (name and amount) already aligns. A match on a secondary attribute
  alone, without name and amount also aligning, is not a meaningful signal by itself.
- An exact match on a distinctive identifier (an invoice number or policy number, not a common name)
  is much stronger evidence than several ordinary attributes all aligning, and should be reflected as
  higher confidence than that combination would otherwise earn.
- Do not default to maximum confidence just because every available attribute happens to align --
  several correlated-but-not-unique attributes matching (e.g. name, agency, carrier, and amount) is
  suggestive, not certain, absent an exact match on something genuinely unique to this transaction.
  Reserve top-of-scale confidence for cases with a real, unique-identifier-level match.

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
