---
name: platform-ui-left-sidebar-navigation
description: >
  Use when the user asks where to find something in the platform or how to get to a page, e.g.
  "How do I create a quote?", "Where can I see my invoices?", "How do I look up a VIN?". Answers
  with the left-sidebar path to the right page.
---

# Left sidebar navigation

The left sidebar is visible on every page, whatever the user is currently doing.

## The one hard rule

Every capability, item, and button in your answer must come from this skill, spelled exactly as
written here. Never from memory. Never invented. Never inferred from what the platform "probably"
has.

- Only these buttons are known: "+ New Business" (All Quotes), "$ Pay Invoices" (Invoices), and
  the option to create a new loss event (Loss Events, All Claims). Don't describe any other
  button, field, or step inside a page.
- If the user asks how to do something inside a page beyond that, give the path to the page and
  say you can't guide them further than that.
- If the user's phrasing doesn't clearly match an entry below or in the glossary, ask what they're
  trying to do. Don't guess a path.

## How to answer

- Every path has the shape `Left sidebar > <Capability> > <Item>`. The capability is one of the
  "###" headings below; the item is one of its bullets.
- Answer in plain language, e.g. "In the left sidebar, click **Accounting**, then **Invoices**."
- If the entry is marked *Restricted*, add that they may not see it, because their account may not
  have permission for it. Don't name a role unless this skill states one.
- Every list shows only the records the user has access to. If the user says a record is missing
  from a list, that may also be a permissions matter.

## Capabilities

Each "###" heading below is one top-level entry in the sidebar. Clicking it expands its items in
place below it, without navigating away from the current page. Each bullet is one of those items.

### Insured
Information about the insureds.
- **All Insured:** A list of all insureds.
- **Driver Lookup:** Search for a driver by any related data and explore every quote, policy, claim and loss event the driver is linked to.
- **VIN Lookup:** Search for a VIN and explore every quote, policy and claim the vehicle is linked to.

### Submissions
Information about submissions grouped by subject.
- **Auto Liability:** A list of all Auto Liability submissions.
- **General Submissions:** A list of all general submissions.
- **Renewals:** A list of all renewal submissions.
- **Endorsements:** A list of all endorsement submissions.
- **Bind Requests:** A list of all bind requests.

### Policies
Information about policies.
- **All Policies:** A list of all policies.

### Claims
Information about claims.
- **Loss Events:** A list of all loss events associated with a claim. Option to create a new loss event.
- **All Claims:** A list of all claims. Option to create a new loss event.
- **Claim Tasks:** A list of all tasks associated with the available claims.
- **Claim Reports:** Access to all reports related to claims.
- **TPAs:** A list of Third Party Administrators. *Restricted.*

### Quotes
Information about quotes.
- **All Quotes:** A list of all quotes. Option to create a new quote with the "+ New Business" button.
- **Quote Tasks:** A list of all tasks associated with the available quotes.
- **New Quote:** Create a new quote for insurance.

To create a quote, point the user to **Quotes > New Quote**. "+ New Business" on All Quotes opens
the same form; mention it only if the user is already on All Quotes or asks about that button.

### Accounting
Information related to accounting.
- **Invoices:** A list of all invoices. Option to pay invoices with the "$ Pay Invoices" button.
- **Payments:** A list of all payments made.
- **Bank Transactions:** A list of all bank transactions. *Restricted.*
- **Reconciliation Workbench:** Bank transactions mapped to the ledger entries they've paid, across every realm — filter by Realm Type and Realm to narrow the view. *Restricted.*
- **Tax Submission Queue:** The queue of tax submissions. *Restricted.*

### Carriers
Information related to carriers.
- **Individual Carriers:** A list of all individual carriers. *Restricted.*
- **Carrier Groups:** A list of all carrier groups. *Restricted.*

### Agencies
Information related to agencies.
- **Manage Agencies:** A list of available agencies. *Restricted.*
- **Reports:** A list of agency reports, such as commission reports.

### Premium Finance
Information related to premium financing. *Restricted:* admin users only.
- **Manage Premium Finance:** A list of premium finance companies.

### User Access
Access to users and user options. *Restricted:* Agency Administrators only.
- **Users:** A list of all users.

## Glossary

Agencies use their own terms. Use this table to match their phrasing to an entry, then answer
with the entry's exact name. A term that isn't here and doesn't clearly match an entry means
asking the user, per the hard rule.

| Agency term | Go to |
|---|---|
| Bill, pay my bill, amount due | Accounting > Invoices ("$ Pay Invoices" to pay) |
| Payment history, receipt, what I've paid | Accounting > Payments |
| Customer, client, account, named insured | Insured > All Insured |
| Driver, driver history, MVR lookup | Insured > Driver Lookup |
| Vehicle, unit, truck, trailer, VIN | Insured > VIN Lookup |
| Application, app, submission | Submissions (ask which kind if unclear) |
| AL, auto, commercial auto | Submissions > Auto Liability |
| Policy change, change request, add/remove a driver or vehicle | Submissions > Endorsements |
| Renewal, renew a policy | Submissions > Renewals |
| Bind, binder, bind order | Submissions > Bind Requests |
| Report a claim, FNOL, first notice of loss, new claim | Claims > Loss Events (create a new loss event) |
| Adjuster company, third-party administrator | Claims > TPAs |
| Quote, proposal, get a price, new business | Quotes > New Quote |
| Market, insurance company, carrier | Carriers > Individual Carriers |
| Commission statement, commission report | Agencies > Reports |
| Finance company, PFC, premium finance company | Premium Finance > Manage Premium Finance |
| Team members, logins, add a user | User Access > Users |
