---
name: ledger-reconciliation-matching
description: >
  Use when proposing a match between an extracted statement line item and an existing, open OURS
  LedgerEntry on the same realm -- before entries are committed and reconciliation links are
  created.
---

Match each confirmed line item against a real, open OURS LedgerEntry for the same realm. An OURS
entry already having one or more matches against it doesn't rule it out -- premium finance
companies routinely split a payment across statements (e.g. $200 today, $800 later) or send
duplicate/overlapping payments for the same event. Matching the same OURS entry more than once is
normal, not an error -- see get_realm_ledger_entries below for how to factor that in.
Submitters vary in what detail they provide (Carrier vs. PFC), so use whatever's available --
insured name and amount are the primary signal; invoice date/number, policy effective date, policy
number, and agency name are secondary.

You'll typically match several line items from one statement in the same turn -- see Tool usage.

**This task is not complete until you call record_disbursement_matches.** Reasoning about the
matches in your response text, without making that call, does not record anything -- no entries
get created and the whole statement fails. Do not stop after gathering candidates and thinking
through the matches; that reasoning is only useful once it's submitted through the tool.

**This skill has no scripts, references, or assets.**

## Matching approach

Compare text attributes (insured/agency/carrier name) by proximity, not exact equality --
abbreviations, punctuation, minor typos, and "LLC" vs "L.L.C." should still count. Weigh by how
close the variation is: a one/two-character difference is much stronger than sharing just a few
words.

Confidence:
- Insured name + amount aligning (even approximately) is the baseline.
- Each additional aligning attribute -- especially policy effective date -- raises confidence
  further, but only on top of an aligning baseline; a secondary attribute alone isn't meaningful.
- An exact match on a distinctive identifier (invoice/policy number) outweighs several ordinary
  attributes aligning.
- Don't default to max confidence just because everything ordinary aligns (name/agency/carrier/
  amount) -- that's suggestive, not certain. Reserve top confidence for a real unique-identifier match.

## Tool usage

Step 1: call get_realm_ledger_entries **exactly once** per statement, before evaluating any line
item -- it takes no arguments and returns every open OURS candidate for this statement's realm.
Reuse that one result for every line item in the turn; the candidate pool doesn't change between
them, and calling it again doesn't get you anything new.

Each candidate includes alreadyMatchedAmount -- how much has already been confirmed-matched
against it. This doesn't rule a candidate out (see above), but use it as context: if a candidate's
own amount is $1000 and alreadyMatchedAmount is $800, a new $200 line completing it is a strong
signal; a new $900 line against that same candidate is worth a note in matchReason (looks like a
duplicate or overpayment), not a reason to withhold the match.

Step 2, mandatory, in the same turn: call record_disbursement_matches exactly once with the whole
batch. This is the only way your matches get recorded -- there is no separate text response that
does this. Include every confirmed line item, even ones with no match (null candidateLedgerEntryId
there), not just the ones you're confident about. You are not finished until this call is made.

Two tool calls total, in that order -- get_realm_ledger_entries once, then record_disbursement_matches
once. No other tool calls are needed to complete this task.

## record_disbursement_matches fields

Per line item:
- lineIndex: the exact lineIndex number the line item was labeled with, not its referenceId --
  referenceId is not guaranteed unique (e.g. a premium line and its taxes/fees line can share the
  same loan reference number), so it can't identify which line item a match is for on its own.
- candidateLedgerEntryId: a real id from get_realm_ledger_entries, or null if nothing plausible --
  never invented. Reusing the same candidate for more than one line item (in this batch or
  matching what's already been matched before, per alreadyMatchedAmount) is fine when the evidence
  supports it -- don't avoid it just because it's already been used.
- confidenceScore: integer 0-100.
- matchReason: brief explanation of the match, or why none was found.

matchStatus is always SUGGESTED, set automatically -- don't set it yourself.
