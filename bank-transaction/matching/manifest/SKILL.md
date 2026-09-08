---
name: bank-transaction-manifest-matching
description: >
  Use when proposing candidate bank transactions for one open Statement/Batch manifest -- the
  reverse direction of direct matching, searching from the manifest side.
---

# Bank transaction manifest matching

Given one manifest (a Statement or Batch, with its own known `declaredTotal`), propose which open
bank transaction(s) on the same realm are its remittance.

**This task is not complete until you call record_manifest_bank_transaction_matches.** Reasoning
about the match in your response text, without making that call, does not record anything.

**This skill has no scripts, references, or assets.**

## Matching approach

Candidates come from `get_unlinked_bank_transactions_for_realm` -- every bank transaction on the
realm not already confirmed-linked to a manifest or directly applied elsewhere, each with
`description`, `amount`, and `date`.

Primary signal: transaction amount against the manifest's `declaredTotal`.
- An exact (or essentially exact) match is the strongest case on its own.
- A Premium Finance Company can remit a statement's total across more than one deposit (e.g. two
  same-day or near-date transactions that together equal the declared total). When no single
  transaction matches the total but two or more plausible ones sum to it, propose all of them
  together, ranked, rather than forcing a single-candidate answer.
- More than one transaction can independently look plausible for the same manifest (e.g. two
  unrelated deposits of similar size) -- propose a ranked list, best first, when genuinely
  ambiguous, rather than only the single best guess.

Secondary, corroborating signal: `date` proximity to the manifest's own document/upload date, and
whether `description` contains anything recognizable (a PFC name, reference number) -- neither is
enough on its own to propose a candidate with no amount evidence behind it.

A transaction whose amount has no plausible relationship to the declared total (alone or combined
with another candidate) is not a match -- leave it out rather than proposing the closest-but-wrong
one.

## Confidence calibration

- **High (80-100):** exact amount match (single transaction, or a combined set that sums exactly)
  with a plausible date.
- **Medium (50-79):** amount is close but not exact, or exact but with a date far from expected.
- **Low (0-49):** a loose amount fit with little else confirming it. If you can't clear even this
  bar, don't propose the candidate.

## Tool usage

Call `get_unlinked_bank_transactions_for_realm` exactly once -- reuse the result for the whole
turn. Then call `record_manifest_bank_transaction_matches` exactly once with your proposal, ranked
best first, from that same candidate list -- never a transaction id you weren't given. Empty is a
valid, expected result when nothing plausible fits; call it with an empty list rather than
guessing.
