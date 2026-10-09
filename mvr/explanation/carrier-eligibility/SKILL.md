---
name: mvr-carrier-eligibility-explanation
description: >
  Use when writing the short overview shown above a results table of which carriers accept a driver
  for one product, from a carrier check computed by code against each carrier's driver rules.
---

# Explaining a carrier check

You get:

- **Carrier check**: `productName`, `asOfDate`, and `carriers`, each with a `verdict`, `reasons` and
  `notes`. Also `externallyCheckedCarriers`, `noDriverChecksCarriers` and `noRulesOnFileCarriers`.
- **Driver**: age, license, status and report date.
- **Conversation so far**.

A table with every carrier, its result and its reasons is shown right below your text. You write the
short overview above it.

## The check is the source of truth

- Never change a verdict, add a carrier, drop a carrier or invent a reason.
- `ACCEPTS`: every driver rule of that carrier passes.
- `ACCEPTS_IF`: nothing declines, but a rule needs a fact the MVR doesn't show. The reasons say what
  to confirm.
- `REFERS`: a rule sends the driver to underwriting review.
- `DECLINES`: at least one driver rule declines. The reasons name it.
- `notes`: driver rules that could not be checked from an MVR, because they depend on quote details,
  are switched off at quote time, or could not be evaluated.
- A reason ending "Checked from the MVR; not enforced at quote time" comes from a suspension or
  MVR-age rule that quoting doesn't check today. Say so when it drives a decline.
- `externallyCheckedCarriers` check eligibility in their own system, so they could not be screened
  here.

## What to write

- 2 to 5 short, plain sentences. No headings, tables or lists of every carrier.
- Start with the counts: how many carriers accept, accept if, refer and decline. Mention only the
  groups that exist.
- Give the main reasons for declines in plain words, grouped, e.g. "Three carriers decline because
  of the 2025 major violation."
- For `ACCEPTS_IF`, say what to confirm, e.g. years of CDL experience, and that confirming it may
  open those carriers.
- If no carrier could be screened, say so and name the ones that check in their own system.
- If the user asked something specific in the conversation, answer it from the check.
