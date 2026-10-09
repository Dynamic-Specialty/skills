---
name: mvr-driver-record-extraction
description: >
  Use when reading an uploaded document that may be a driver's Motor Vehicle Record (MVR), to record
  what it shows (document type, driver, license, status and driving history) through
  record_mvr_reading.
---

# Reading a Motor Vehicle Record (MVR)

You read one uploaded document and record what it shows by calling `record_mvr_reading` once.

You report what is printed. You do not decide whether the MVR is valid, and you do not decide
whether the driver is acceptable. Code decides both from what you record.

## Never guess

- If a field isn't printed, or you can't read it, leave it null. For the yes/no and status fields,
  use `UNKNOWN`.
- Copy names, numbers and descriptions exactly as printed.
- Write every date as `YYYY-MM-DD`. Convert printed formats: `10/01/2026` becomes `2026-10-01`.
  If only a month and year are printed, use the first day of that month.

## What the document is (`documentType`)

- `MVR`: an official driving-history report for a driver. It is issued by a state motor vehicle
  agency (DMV, DPS, BMV, MVD, RMV...) or by a vendor reporting that agency's data. It shows the
  license details and a history section listing violations, accidents and suspensions, or stating
  there are none.
- `DRIVER_LICENSE`: the license card itself.
- `INSURANCE_CARD`, `VEHICLE_REGISTRATION`.
- `LOSS_RUN`: an insurance claims history.
- `POLICE_REPORT`: a crash or incident report.
- `OTHER`: anything else.

When the document is not an MVR, record its type and whatever driver details are visible. Leave the
rest null and the incidents empty.

## Report fields

- `issuingSource`: the agency or vendor named on the report.
- `reportDate`: the date the report was produced ("Report date", "Date ordered", "Run date",
  "Printed on"). Never the license issue or expiration date.
- `driverCount`: how many different drivers the document covers. Normally 1.
- `historySectionPresent`: true when there is a history section, including one that says there is
  nothing to report ("No violations", "Clear record").
- `pagesComplete`: false only when a page is obviously missing, e.g. "Page 2 of 3" with page 2
  absent, or a section cut off mid-way.

## Driver and license

- `driverName`, `dateOfBirth`, `licenseNumber`: as printed.
- `licenseState`: the issuing state as a two-letter code.
- `licenseClass`: as printed, e.g. `A`, `CDL-A`, `Class C`.
- `commercialLicense`:
  - `YES` only when the report labels the license as commercial or a CDL ("CDL", "Commercial",
    "CDL Class A").
  - `NO` when it shows a regular, non-commercial license, or only a commercial learner's permit.
  - `UNKNOWN` when it doesn't say. A class letter alone is not enough. In most states Class C is the
    regular car license; it is a CDL only when the report calls it commercial.
- `commercialIssueDate`: the date the CDL was first issued, only when the report labels it as the
  commercial license's date ("CDL original issue", "Commercial issue date"). The general license
  issue date is not it.
- `licenseStatusAsPrinted`: the current status exactly as printed.
- `licenseStatus`: the current status, normalized:
  - `VALID`: valid, active, clear, licensed, eligible.
  - `SUSPENDED`, `REVOKED`, `CANCELLED`, `EXPIRED`.
  - `DISQUALIFIED`: a commercial disqualification.
  - `OTHER`: anything else, e.g. "Restricted", "Surrendered".
  - `UNKNOWN`: not shown.
  - Use the status today. A suspension that has ended is a history entry, not the current status.

## Driving history (`incidents`)

List every entry in the history section once, oldest first. Points totals, summaries and code
legends are not entries.

- `kind`:
  - `VIOLATION`: a conviction or citation for a moving traffic offense.
  - `ACCIDENT`: a reported crash.
  - `SUSPENSION`: a suspension, revocation, cancellation or disqualification action, including ones
    that have ended.
  - `OTHER`: anything else: non-moving offenses (parking, equipment, registration, insurance
    paperwork), reinstatements, administrative entries.
- `date`: the date of the offense or accident. If only a conviction date is printed, use it. Use a
  posting date only when nothing else is printed.
- `description`: as printed, including any code.
- `severity`, for violations only:
  - When the report itself classes the violation (e.g. "Major", "Serious", a points category that
    the report labels major), follow the report.
  - `MAJOR`: driving under the influence of alcohol or drugs (DUI, DWI, OWI); refusing a chemical
    test; reckless driving; leaving the scene of an accident; driving while suspended, revoked,
    cancelled or disqualified; vehicular homicide, manslaughter or assault; a felony involving a
    vehicle; fleeing or eluding police; racing; speeding 25 mph or more over the limit; driving a
    commercial vehicle without a CDL.
  - `MINOR`: every other moving violation, e.g. speeding under 25 mph over, failure to yield,
    running a red light or stop sign, improper lane change, following too closely, improper passing.
  - `UNKNOWN`: when the entry doesn't let you tell, e.g. a bare state code with no text or legend.
- `atFault`, for accidents only:
  - `YES` when the report marks the driver at fault or chargeable, or ties a conviction to the
    accident.
  - `NO` when it marks the accident not at fault or non-chargeable.
  - `UNKNOWN` when it doesn't say. Most MVRs don't.
- The same event can appear twice, e.g. as a citation and as its conviction. List it once.

## Recording

Call `record_mvr_reading` exactly once with everything above. If it is rejected, fix what the
message says and call it again.
