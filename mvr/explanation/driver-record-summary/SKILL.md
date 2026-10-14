---
name: mvr-driver-record-summary
description: >
  Use when writing the short plain-language summary shown above a driver's MVR facts in the
  Underwriting chat, from an MVR check already computed and summarized by code.
---

# Summarizing an MVR check

A list of the driver's facts and every incident is shown right below your text. You write a short
summary above it for an underwriter. Never repeat the list.

## What the check contains

- `valid` and `issues`: whether the document is a usable MVR, and why not.
- `documentType`, `driverName`, `ageYears`, the license fields and `licenseStatus`.
- `lastThreeYears`: counts code already worked out. A range such as "0 to 1" means some incident
  details are missing.
- `majorViolationsLast3Years` and `atFaultAccidentsLast3Years`: those entries, as printed.
- `incidentsMissingDetails`: incidents whose severity, fault or date the MVR doesn't show.

## Rules

- Use only the check. Never add facts, never recount incidents, never do date arithmetic: the counts
  are already worked out.
- Never repeat the date of birth or the license number.
- Don't say which carriers accept the driver. That comes after the user picks a product.
- Don't comment on how old the report is. It doesn't decide whether a carrier accepts the driver,
  and its date is already in the fact list.
- Write 2 to 4 short, plain sentences.

## What to say

- When `valid` is false: say plainly that the document can't be used and why, from `issues`.
  Suggest uploading a clearer or complete copy when that would help.
- When `valid` is true: say whether the record is clean. Then name what matters most, in this order:
  - a license that is suspended, revoked, cancelled, disqualified or expired;
  - major violations and at-fault accidents in the last three years;
  - recent minor violations;
  - missing details: CDL status, date of birth, or incidents with unknown fault or severity.
- If the conversation mentions something relevant, such as the product the user cares about, you may
  refer to it briefly.

## Handing in your text

Call `record_text` exactly once with your finished summary. It accepts plain sentences only: no
tables, no lists, no headings, nothing long. If it rejects your text, fix what it says and call it
again.
