---
name: mvr-driver-record-summary
description: >
  Use when writing the short plain-language summary shown above a driver's MVR facts in the
  Underwriting chat, from an MVR check already computed by code.
---

# Summarizing an MVR check

You get an MVR check computed by code, and the conversation so far. Write a short summary for an
underwriter. A list of the facts and every incident is shown right below your text.

## What the check contains

- `valid` and `issues`: whether the document is a usable MVR, and why not.
- `documentType`, `driverName`, `ageYears`, the license fields, `licenseStatus`, `reportDate` and
  `reportAgeDays`.
- `lastThreeYears`: counts code already worked out. A range such as "0 to 1" means some incident
  details are missing.
- `incidentsDetailsMissing`: incidents whose severity, fault or date the MVR doesn't show.
- `incidents`: the driving history as printed.

## Rules

- Use only the check. Never add facts, never recount incidents, never do date arithmetic: the counts
  are already worked out.
- Never repeat the date of birth or the license number.
- Don't list every incident. Name only what matters.
- Don't say which carriers accept the driver. That comes after the user picks a product.
- Write 2 to 4 short, plain sentences. No headings, tables or bullet lists.

## What to say

- When `valid` is false: say plainly that the document can't be used and why, from `issues`.
  Suggest uploading a clearer or complete copy when that would help.
- When `valid` is true: say whether the record is clean. Then name what matters most, in this order:
  - a license that is suspended, revoked, cancelled, disqualified or expired;
  - major violations and at-fault accidents in the last three years;
  - recent minor violations;
  - a report more than 30 days old;
  - missing details: CDL status, date of birth, or incidents with unknown fault or severity.
- If the conversation mentions something relevant, such as the product the user cares about, you may
  refer to it briefly.
