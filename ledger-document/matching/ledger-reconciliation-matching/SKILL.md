---
name: ledger-reconciliation-matching
description: >
  Use when proposing a match between a saved statement line and an open OURS ledger entry on the
  same realm, before a person accepts or rejects the suggestion.
---

Match statement lines against open OURS ledger entries on the same realm. You only suggest. A
person accepts or rejects each suggestion.

## How to answer

1. Call get_realm_ledger_entries once, before evaluating any line. It takes no arguments and
   returns the open OURS entries (the candidates) that still have a balance left.
2. Call record_disbursement_matches once, with only the lines you suggest a match for. Leave out
   lines with no plausible match, and lines you would score below 65. A person matches those by
   hand. If no line qualifies, call it with an empty list.

Reasoning in text records nothing. The task is complete only when step 2 is called.

Each candidate has alreadyMatchedAmount: what is already confirmed against it. The remaining
balance is its amount minus alreadyMatchedAmount. Several statement lines may match one candidate
(for example a split payment) while it has balance left.

## Evidence

A suggestion needs a baseline and at least one confirming signal.

Baseline, either of:
- The names align: insured name plus agency name. Compare by proximity, not equality.
  Abbreviations, punctuation and minor typos count. A one or two character difference is much
  stronger than sharing a few words. Names alone are never enough, because one insured and agency
  can have several unrelated policies.
- An identifier (policy or invoice number) literally appears on both the line and the candidate.
  A reference that exists only on the statement is not an identifier match. Do not cite it as one.
  A shared identifier is the strongest evidence. It can carry a match even if date or amount is
  imperfect.

Confirming signals:
- Date: the policy effective date is within a few days of the candidate's, not weeks. A wide gap
  is a negative signal.
- Amount, compared with the remaining balance:
  - Equal, or essentially equal: confirms on its own.
  - A real fraction (a third, a half): a plausible partial payment. Confirms more weakly. Say so
    in matchReason.
  - A token fraction with nothing else confirming: not a confirming signal.
  - More than the remaining balance: a possible overpayment. Not a confirming signal. Lean on date
    or identifier. If you still suggest it, say in matchReason that it exceeds the balance.
- Several lines in this batch on one candidate: judge their amounts together. Totals at or near
  the remaining balance support each other. Totals well short or well over lower the confidence of
  all of them.

Without a baseline, nothing else justifies a match, however many dates, amounts, line types or
reason codes line up. Line type and reason code are never evidence for a match. They only rule a
candidate out or lower the score.

## Confidence

- Baseline plus one weak signal (a loose date, or a small fraction of the balance): low. Do not
  round it up. A close date with nothing else confirming also belongs here.
- Baseline plus a close date plus a plausible full or partial amount: high, close to an identifier
  match but not equal to it.
- An exact identifier match: can be high even if amount or date is imperfect.
- Top confidence is for a real identifier match, or a full amount match with a close date.

Floor: if your confidence for a line is below 65, leave it out. A weak guess costs the reviewer
more than a missing suggestion.

## Carrier statements (realm type CARRIER)

These rules replace the name and agency baseline above. Carrier lines and candidates have no
agency, so ignore agency.

- Baseline: the policy number, with the insured name. Compare policy numbers ignoring spacing and
  punctuation.
  - Equal policy number: baseline, even if the insured name is spelled differently.
  - Policy numbers that differ by one or two characters (a typo, a swapped digit) are close. A
    close policy number plus an insured name that aligns is a baseline, weaker than an equal one.
    Say in matchReason that the policy numbers differ.
  - A matching insured name with a policy number that is not close: no baseline.
  - A policy number and insured name that both differ: no baseline.
- Line type: a line matches only a candidate with the same itemType (PREMIUM with PREMIUM,
  CARRIER_COMMISSION with CARRIER_COMMISSION, PREMIUM_RETURN with PREMIUM_RETURN, and so on). Never
  match a premium line to a commission candidate or the reverse. Ignore the candidate's entryType.
- Amounts are positive on both sides. The line type carries the direction. Compare amounts as they
  are.
- Several lines, one candidate: a policy can have more than one line of one type, for example a
  premium line plus a second premium line that earns no commission. Their amounts together can
  equal one candidate. A line that is only a fraction of the balance counts only when the lines
  that make up the rest belong to the same policy. A fraction is never evidence for a line of
  another policy.
- Reason code (BIND, RENEWAL, ENDORSEMENT, CANCELLED, REINSTATEMENT): it should agree with the
  candidate's. A mismatch lowers the confidence.

## record_disbursement_matches fields

Per line:
- lineIndex: the exact lineIndex the line was labeled with, not its referenceId. The referenceId
  is not unique: a premium line and its taxes and fees line can share one.
- candidateLedgerEntryId: a real id from get_realm_ledger_entries. Never invented. One candidate
  may serve several lines when the evidence supports it.
- confidenceScore: integer 0 to 100.
- matchReason: one short sentence naming only the signals that drove the decision, for example
  "date within 2 days, amount matches remaining balance". Do not recap everything you checked.
  Never refer to a line by its lineIndex or as "line N": that number only exists inside this
  batch. Describe the line instead, for example "together with the policy's other premium line,
  the amounts equal the remaining balance". Say a field matches only when the two values are
  equal. Never write that a reason code, line type, date or policy number matches when they
  differ.

matchStatus is always SUGGESTED and is set automatically.
