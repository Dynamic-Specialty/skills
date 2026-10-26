---
name: policy-helper-poi-identification
description: Reading which policy a scanned or photographed Proof of Insurance (POI) is for, so the system can find it in Dynamic's records.
---

# Reading a POI's identifying fields

You are given images of the first pages of a document a user uploaded. Read only what is printed.
The system uses your reading to look the policy up in Dynamic's records, so an exact copy of the
printed text matters more than anything else.

## First: is it a proof of insurance?

Set `proofOfInsurance` to true only for a proof, certificate or evidence of insurance. A policy
wording, an invoice, an MVR or anything else is false — record it and stop.

## Fields

Dynamic's POIs print these at the top of page 1:

- **General Agency** — e.g. "DYNAMIC ROOT".
- **Policy No** — copy it exactly, with dashes. It's often shared by many insureds; that's fine.
- **Certificate Period** — two dates, "MM/DD/YYYY - MM/DD/YYYY". Record them as YYYY-MM-DD.
- **Carrier** — e.g. "LLOYDS OF LONDON".

Further down:

- **Coverage** — the line in capitals under "Coverages", e.g. "PHYSICAL DAMAGE", "MOTOR CARGO",
  "NON-TRUCKING LIABILITY". Copy it as printed.
- **Certificate holder** — the insured's name under "Certificate Holder" (not the broker).
- **VINs** — every VIN in "Schedule of Covered Autos". VINs are 17 characters and never contain
  the letters I, O or Q: a character that looks like O is a zero, like I is a one.

## Rules

- Leave a field null when it isn't printed or you can't read it. Never guess.
- Don't record dates of birth, license numbers or addresses. They aren't needed.
- If a submission is rejected, re-read the page for the field named in the reason, then call
  `record_poi_identification` again.
