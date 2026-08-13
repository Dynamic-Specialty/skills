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

Baseline: insured name + agency name aligning. Compare by proximity, not exact equality --
abbreviations, punctuation, and minor typos still count, and a one/two-character difference is
much stronger evidence than sharing just a few words.

Name alignment alone never justifies a match -- the same insured/agency pair can legitimately
appear on multiple, unrelated policies. Require at least one confirming signal on top of it:

- **Date**: policy effective date within a few days of the OURS entry's -- not weeks. A wide gap
  is a *negative* signal (likely a different policy period), not a loose match.
- **Identifier**: a policy or invoice number that literally appears on both the statement line and
  the OURS entry. A reference/loan number that only exists on the statement side, with nothing on
  the OURS entry to compare it to, is not an identifier match -- don't cite it as if it were.
- **Amount**: compare the line's amount to the candidate's remaining balance (its amount minus
  alreadyMatchedAmount, from get_realm_ledger_entries):
  - Equal (or essentially so) to the remaining balance -- confirms the baseline on its own.
  - A real fraction of the remaining balance (e.g. a third, a half) -- plausible partial payment,
    confirms the baseline but more weakly than a full match; say so in matchReason.
  - A small fraction of the remaining balance (a token amount relative to it) with nothing else
    confirming -- too weak on its own, treat as informational, not confirming.
  - More than the remaining balance -- possible overpayment, don't treat as confirming; flag it
    and lean on date/identifier instead.
  When more than one statement line in this batch targets the same candidate, judge their amounts
  together: combined totals that land at or near the remaining balance support each other;
  combined totals well short of or well past it should lower confidence for all of them.

No confirming signal at all means don't propose the match, regardless of how well the names align.

Confidence:
- Baseline + exactly one weak confirming signal (a loose-but-not-tight date, or a small fraction
  of remaining balance) belongs in the lower half of the range -- don't round this up.
- Baseline + a close date + a plausible (full or partial) amount supports moderate-to-high
  confidence.
- An exact identifier match (policy or invoice number literally shared) is the strongest signal --
  it can carry high confidence on its own even if amount or date is imperfect.
- Reserve top confidence for a real identifier match, or a full/near-full amount match alongside a
  close date -- not just because every ordinary attribute happens to align.

## Tool usage

Step 1: call get_realm_ledger_entries **exactly once** per statement, before evaluating any line
item -- it takes no arguments and returns every open OURS candidate for this statement's realm.
Reuse that result for the whole turn; calling it again returns nothing new.

Each candidate includes alreadyMatchedAmount -- how much has already been confirmed-matched
against it. Subtract it from the candidate's amount to get the remaining balance used in Matching
approach above: a candidate with amount $1000 and alreadyMatchedAmount $800 has a $200 remaining
balance, so a new $200 line is a strong completion; a new $900 line against that same candidate
exceeds it and is worth flagging as a likely duplicate or overpayment in matchReason, not
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
- matchReason: one short sentence, citing only the signal(s) that actually drove the decision
  (e.g. "date within 2 days, amount matches remaining balance" or "no candidate within a
  plausible date range") -- not a recap of every signal you checked, and not your reasoning
  process.

matchStatus is always SUGGESTED, set automatically -- don't set it yourself.
