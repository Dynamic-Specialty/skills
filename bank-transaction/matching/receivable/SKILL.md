---
name: bank-transaction-receivable-matching
description: >
  Use when proposing candidate bank transactions for one specific open receivable -- the reverse
  direction of direct matching, searching from the receivable side.
---

# Bank transaction receivable matching

Given one specific open receivable (its own `amount` and `insuredName` provided to you), propose
which open bank transaction(s) on the same realm could be applied to pay it.

**This task is not complete until you call record_receivable_bank_transaction_matches.** Reasoning
about the match in your response text, without making that call, does not record anything.

**This skill has no scripts, references, or assets.**

## Matching approach

Candidates come from `get_unlinked_bank_transactions_for_receivable` -- already pre-filtered to
transactions eligible for direct application, and already sorted closest-by-amount-first against
this receivable, capped to the most plausible handful. This is not the realm's entire transaction
history; treat every candidate returned as a serious contender worth evaluating, not a long list
to skim for the obvious top pick.

Primary signal: candidate `amount` against the receivable's own `amount`.
- An exact (or essentially exact) match is the strongest case, especially for the first (closest)
  candidate in the list.
- A candidate's `description` naming or closely matching the receivable's `insuredName` is strong
  corroborating evidence, particularly when more than one candidate has a similarly close amount.
- More than one candidate can plausibly fit (e.g. two deposits of very similar size) -- propose a
  ranked list, best first, rather than only the single best guess, when genuinely ambiguous.

A candidate whose amount is meaningfully different from the receivable's amount, with nothing else
confirming it, is not a match -- since candidates are already sorted by proximity, if even the
closest one doesn't hold up, the right answer is usually an empty result, not stretching to the
next-closest.

## Confidence calibration

- **High (80-100):** exact or near-exact amount match, reinforced by a name match in the
  description.
- **Medium (50-79):** amount is close but not exact, or exact with no corroborating name evidence.
- **Low (0-49):** a loose amount fit with nothing else confirming it. If you can't clear even this
  bar, don't propose the candidate.

## Tool usage

Call `get_unlinked_bank_transactions_for_receivable` exactly once -- reuse the result for the
whole turn. Then call `record_receivable_bank_transaction_matches` exactly once with your
proposal, ranked best first, from that same candidate list -- never a transaction id you weren't
given. Empty is a valid, expected result when nothing plausible fits; call it with an empty list
rather than guessing.
