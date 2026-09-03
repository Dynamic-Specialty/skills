---
name: bank-transaction-triage
description: Classifying a bank transaction into its Category (Agency or PFC) and the specific Realm it belongs to
---

# Bank transaction triage

You are given a batch of bank transactions synced from Plaid. For each one, propose:

- **category** — `AGENCY` if the transaction was originated by (or is a payment from) a retail
  agency, `PFC` if it's a payment from a Premium Finance Company, or `UNCATEGORIZED` if you
  genuinely cannot tell.
- **selectedRealmId** — the id of the specific Realm this transaction belongs to, chosen from that
  transaction's own candidate list (`pfcCandidates` if you chose `PFC`, `agencyCandidates` if you
  chose `AGENCY`). Never invent an id that isn't in that transaction's own candidate list — if
  nothing on the list is a confident match, leave this null rather than guessing.
- **confidenceScore** (0-100) — your overall confidence in the proposal as a whole (category and,
  if proposed, the selected entity together).
- **reason** — a short, factual explanation a human reviewer can act on without having to re-derive
  your reasoning (e.g. "description matches PFC candidate 'IPFS Corp' by abbreviated name").

## Reading the transaction description

The `description` field is a raw ACH/bank transaction description, not a clean company name — it
often includes originator codes, truncated names, reference numbers, or generic banking boilerplate
mixed in with the actual payer/payee name. Extract the meaningful part before matching it against
candidates.

## Category signals

**PFC-leaning signals:**
- The description names or clearly abbreviates a known premium finance company (e.g. "IPFS",
  "FIRST INSURANCE", "AFCO", "IPFC", "PREMIUM FINANCE", "PREM FIN").
- The description includes financial-institution-style language associated with installment
  lending (e.g. "LOAN", "FINANCE CO", "INSTALLMENT") in combination with insurance-related wording.
- The transaction recurs at a regular cadence with a similar amount — premium finance payments are
  typically scheduled installments, unlike one-off agency disbursements. Treat this as a supporting
  signal only; never classify on cadence/amount alone without corroborating description evidence.

**AGENCY-leaning signals:**
- The description names a specific retail agency, or a payment-processor style code associated with
  agency billing (e.g. "EPAY", "AGENCY PMT", "RETAIL PAY").
- The description matches (exactly or closely) the name or abbreviated name of one of the
  transaction's own `agencyCandidates`.

**When to stay UNCATEGORIZED:**
- The description is too generic or garbled to support either category with real confidence (e.g.
  a bare reference number, a generic "TRANSFER" or "ACH CREDIT" with no identifying name).
- Don't force a category to avoid leaving one null — an honest `UNCATEGORIZED` with a low
  `confidenceScore` is far more useful to a reviewer than a confident-sounding guess that's wrong.

## The ruleHint signal

Each transaction includes a `ruleHint` — the output of a simple, literal substring-matching
classifier (looks for exact strings like "IPFS", "FIRST INSURANCE", "EPAY" in the description). Use
it as one input, not the final answer:

- If `ruleHint` agrees with your own reading of the description, that's corroborating evidence —
  raise your confidence accordingly.
- If `ruleHint` says `UNCATEGORIZED`, don't treat that as evidence the transaction has no category —
  it just means none of that classifier's fixed strings appeared; use your own judgment on the full
  description.
- If `ruleHint` disagrees with your own reading, trust your own reading of the actual description
  and candidate lists — the rule list is a small, fixed set of known patterns, not authoritative.

## Selecting the entity

Once you've picked a category, look at that category's candidate list for this transaction
(`pfcCandidates` or `agencyCandidates` — never cross-reference the other list). Each candidate has an
`id`, `name`, and `abbreviatedName`.

- Prefer an exact or near-exact match between the description and a candidate's `name` or
  `abbreviatedName`.
- `agencyCandidates` is a shortlist (not exhaustive) produced by searching agency names against the
  transaction's description — if the right agency simply isn't among the candidates given, leave
  `selectedRealmId` null and explain that in `reason`, rather than picking the closest-sounding
  wrong one.
- `pfcCandidates` is the full list of active PFC realms, so a missing match there is more likely a
  genuine "this isn't actually a PFC transaction" signal than a shortlisting gap — weigh that when
  deciding your category, not only your entity choice.

## Confidence calibration

- **High (80-100):** a clear, specific name match to exactly one candidate, corroborated by
  `ruleHint` or unambiguous description wording.
- **Medium (50-79):** a plausible category and/or entity match, but based on partial/ambiguous
  description text, or a name match that isn't exact.
- **Low (0-49):** you're guessing more than concluding — a generic description, no real candidate
  match, or conflicting signals. A low score here is what routes the proposal to human review, so
  don't inflate it to avoid that outcome.

## Writing the reason

State the concrete evidence, not just the conclusion — a reviewer should be able to glance at your
reason and immediately see whether it holds up. Reference the specific wording or candidate name
that drove the decision (e.g. "Description contains 'IPFS' and ruleHint agrees; matched PFC
candidate id 42 (IPFS Corp) by abbreviated name" rather than just "This looks like a PFC payment").

## Output format

Return exactly one result per transaction in the batch, echoing back its `batchIndex` and
`transactionId` exactly as given — these are used to match your answer back to the transaction it's
about, not for your own reasoning.
