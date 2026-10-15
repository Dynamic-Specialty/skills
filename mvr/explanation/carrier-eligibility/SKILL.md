---
name: mvr-carrier-eligibility-explanation
description: >
  Use when writing the short overview shown above a results table of which carriers accept a driver
  for one product, from a carrier check already computed and summarized by code.
---

# Explaining a carrier check

A table with every carrier, its result and its reasons is shown right below your text. You write the
short overview above it. Never repeat the table.

## What you get

- **Carrier check**, already summarized by code:
  - `product`.
  - `counts`: how many carriers accept, accept if, refer and decline.
  - `declineReasons` and `referralReasons`: each reason, with the carriers it applies to.
  - `toConfirm`: what has to be confirmed before the "accept if" carriers accept, with the carriers
    each item applies to.
  - `checkedInOwnSystem`, `noDriverChecks`, `noRulesOnFile`: carriers that could not be screened here.
  - `pointsOfAttention`: things an underwriter should review that no carrier rule decides, such as
    past suspensions. They are shown under the table and never change a carrier's result.
  - `driver`: age, license and status.
- **Conversation so far**.

## The check is the source of truth

- Never change a count, add or drop a carrier, or invent a reason.
- A reason ending "Checked from the MVR; not enforced at quote time" comes from a suspension rule
  that quoting doesn't check today. Say so when it drives a decline.
- How old the MVR is never changes a result, and a note under the table already covers it. Don't
  mention the MVR's age unless the user asks about it.

## What to write

- 2 to 5 short, plain sentences.
- Start with the counts: how many carriers accept, accept if, refer and decline. Mention only the
  groups that exist.
- Give the main decline reasons in plain words, with how many carriers each one affects, e.g. "Three
  carriers decline because of the 2025 major violation." Name carriers only when there are one or
  two.
- For "accept if", say what to confirm, e.g. years of CDL experience, and that confirming it may
  open those carriers.
- Don't repeat the points of attention: they are already shown under the table. Never present
  them as a reason a carrier declines.
- If no carrier could be screened, say so and name the ones that check in their own system.
- If the user asked something specific in the conversation, answer it from the check.

## Handing in your text

Call `record_text` exactly once with your finished text. It accepts plain sentences only: no tables,
no lists, no headings, nothing long. If it rejects your text, fix what it says and call it again.
