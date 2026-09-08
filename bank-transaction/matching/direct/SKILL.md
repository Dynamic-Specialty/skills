---
name: bank-transaction-direct-matching
description: >
  Use when proposing candidate matches for one bank transaction -- direct open receivables it
  could be applied against, and/or one open Statement/Batch manifest it could be linked to.
---

# Bank transaction direct matching

Given one bank transaction, propose two independent kinds of candidate, either or both:

- **Direct entry matches** -- one or more open OURS receivables this transaction's amount pays,
  in whole or in part.
- **Manifest matches** -- one or more open Statements/Batches this transaction's amount could be
  the remittance for.

These are genuinely separate questions about the same transaction, not a single choice between
them -- propose whichever kind(s) the evidence actually supports. It's normal for one half to be
empty while the other has candidates, or for both to have candidates, or for neither to.

**This task is not complete until you call record_matches.** Reasoning about the match in your
response text, without making that call, does not record anything.

**This skill has no scripts, references, or assets.**

## Direct entry matching approach

Candidates come from `get_unsettled_ledger_entries_for_realm` -- every open OURS receivable on
the transaction's realm, each with `amount`, `alreadyAppliedAmount`, and `remainingAmount`
(amount minus alreadyAppliedAmount). A candidate with a nonzero `alreadyAppliedAmount` is still a
valid target -- partial and split payments against the same receivable are normal, not a reason
to skip it.

Primary signal: the transaction's own `description` naming (or closely matching) a candidate's
`insuredName`. Bank descriptions are often abbreviated or contain extra routing text -- match on
the meaningful part, not literal equality.

Confirming signal: the transaction amount against one candidate's `remainingAmount` --
- Exact (or essentially exact) match to one candidate's remaining balance is the strongest case.
- A transaction amount that doesn't match any single candidate's remaining balance, but does match
  the **sum** of two or more plausible candidates' remaining balances combined (e.g. the same
  insured has two open receivables, and this deposit covers both at once), is a real, expected
  scenario -- propose all of them together in `entryMatches`, not just the closest single one or
  none at all. Don't default to a single-candidate answer just because it's simpler; a split
  payment across several receivables is exactly as valid an outcome as a single match.
- Never propose a set of candidates whose remaining balances, summed, meaningfully exceed the
  transaction amount -- that's not a plausible split.

Name alignment alone, with no amount evidence at all (no single or combined match), is too weak to
propose -- leave `entryMatches` empty rather than guessing off name alone.

## Manifest matching approach

Candidates come from `get_realm_manifests_for_realm` -- every open Statement/Batch on the realm
not already fully tallied, each with `declaredTotal` and `allLinesConfirmed` (whether every
receivable under that manifest already has a confirmed statement-line match).

- An exact (or essentially exact) match between the transaction amount and a manifest's
  `declaredTotal`, with `allLinesConfirmed` true, is a clean, high-confidence candidate -- the
  manifest is fully accounted for internally and this transaction is very likely its remittance.
- An exact amount match with `allLinesConfirmed` false is still worth proposing, but at lower
  confidence -- the total lines up, but not every line under it has a home yet, so linking it
  won't fully settle immediately.
- More than one manifest can plausibly fit (e.g. two same-day statements with similar totals) --
  propose a ranked list, best first, rather than only the single best guess.
- A transaction amount with no manifest total anywhere close is not a manifest match -- leave
  `manifestMatches` empty rather than proposing the closest-but-still-wrong one.

## Confidence calibration

- **High (80-100):** exact amount match (single candidate or a combined split) plus a real name
  alignment, or an exact manifest-total match with `allLinesConfirmed` true.
- **Medium (50-79):** amount matches but name evidence is weak/absent, or a manifest total matches
  exactly but `allLinesConfirmed` is false.
- **Low (0-49):** a plausible but inexact amount fit with some corroborating evidence. If you can't
  clear even this bar, don't propose the candidate -- leave it out rather than including a
  low-confidence guess.

## Tool usage

Call `get_unsettled_ledger_entries_for_realm` and `get_realm_manifests_for_realm` as needed, each
exactly once -- reuse the result for the whole turn, don't call either again. Then call
`record_matches` exactly once with your proposal: `entryMatches` (empty if no direct candidate
fits, otherwise every candidate you're proposing -- one or several), and `manifestMatches` (empty
if none fit, otherwise a ranked list, best first). If nothing plausible fits either path, call
`record_matches` with both arguments empty rather than guessing.
