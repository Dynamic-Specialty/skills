---
name: ledger-reconciliation-matching
description: >
  Use when proposing a match between an extracted statement line item and an existing, open OURS
  LedgerEntry on the same realm -- before entries are committed and reconciliation links are
  created.
---

Match each confirmed line item against a real, open OURS LedgerEntry for the same realm.

An OURS entry with existing matches is still a valid candidate. PFCs routinely split a payment
across statements (e.g. $200 today, $800 later) or send duplicate/overlapping payments for the
same event -- matching the same OURS entry more than once is normal, not an error.

You'll typically match several line items from one statement in the same turn -- see Tool usage.

**This task is not complete until you call record_disbursement_matches.** Reasoning about the
matches in your response text, without making that call, does not record anything -- no entries
get created and the whole statement fails.

**This skill has no scripts, references, or assets.**

## Matching approach

Baseline: insured name + agency name, plus policy effective date or policy number -- whichever the
statement provides, since policy number isn't always present. Compare names by proximity, not
exact equality: abbreviations, punctuation, and minor typos still count, and a one/two-character
difference is much stronger evidence than sharing just a few words.

Amount is not part of the baseline. A genuine partial or duplicate payment won't align with the
OURS entry's amount, so don't require alignment before considering a candidate. Use amount instead
as a confidence modifier: full alignment, or a partial that plausibly completes the remaining
balance (OURS amount minus the candidate's alreadyMatchedAmount), raises confidence; a mismatch
that fits neither pattern lowers confidence but doesn't disqualify the candidate -- note it in
matchReason (e.g. possible overpayment).

Confidence:
- Baseline aligning, even approximately, is the floor.
- Each additional aligning attribute raises confidence further, especially a clean amount match.
- An exact match on a distinctive identifier (invoice or policy number) outweighs several ordinary
  attributes aligning.
- Don't default to max confidence just because everything ordinary aligns -- reserve top
  confidence for a real unique-identifier match.

## Tool usage

Step 1: call get_realm_ledger_entries **exactly once** per statement, before evaluating any line
item -- it takes no arguments and returns every open OURS candidate for this statement's realm.
Reuse that result for the whole turn; calling it again returns nothing new.

Each candidate includes alreadyMatchedAmount -- how much has already been confirmed-matched
against it. Use it to judge a new match's amount: a candidate with amount $1000 and
alreadyMatchedAmount $800 makes a new $200 line a strong completion, while a new $900 line against
that same candidate is worth flagging as a likely duplicate or overpayment in matchReason, not
withholding.

Step 2, mandatory, same turn: call record_disbursement_matches exactly once with the whole batch.
This is the only way matches get recorded. Include every confirmed line item, even ones with no
match (null candidateLedgerEntryId), not just the ones you're confident about.

Two tool calls total, in that order -- nothing else is needed to complete this task.

## record_disbursement_matches fields

Per line item:
- lineIndex: the exact lineIndex the line item was labeled with, not its referenceId --
  referenceId isn't guaranteed unique (e.g. a premium line and its taxes/fees line can share the
  same loan reference number).
- candidateLedgerEntryId: a real id from get_realm_ledger_entries, or null if nothing plausible --
  never invented. Reusing the same candidate across multiple line items is fine when the evidence
  supports it.
- confidenceScore: integer 0-100.
- matchReason: brief explanation of the match, or why none was found.

matchStatus is always SUGGESTED, set automatically -- don't set it yourself.
