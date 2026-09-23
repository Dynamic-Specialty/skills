---
name: agent-main-ui-sidebar-view
description: Use when the user asks for guidance on any of the platform capabilities that are not from your specialization. e.g. "How do I create a Quote?" or "Where can I see my invoices?". Use the correct section of this skill to guide the user through the UI.
---

# Left sidebar navigation

The left sidebar is always visible, on every page, regardless of what context the user is
currently in. This skill is the same for every agent — it is not specific to one wizard or chat
context.

## How to answer

- Every path has this shape: "Left sidebar > \<Capability\> > \<item\>" — the capability is one
  of the "###" headings below, the item is one of its bullets.
- Give the exact path as that sequence, e.g. "Left sidebar > Insured > VIN Lookup".
- If a capability or item needs a specific role or permission to see, say so.
- If the user's phrasing doesn't clearly match an entry below, ask what they're trying to do
  instead of guessing a path.

## Capabilities

Each "###" heading below is one top-level entry in the sidebar. Clicking it expands its items in
place below it, without navigating away from the current page. Each bullet is one of those items.

### Insured
Information about all the insureds accessible by the user.
- **All Insured:** A list of all insureds.
- **Driver Lookup:** Search for a driver by any related data and explore every quote, policy, claim and loss event the driver is linked to.
- **VIN Lookup:** Search for a VIN and explore every quote, policy and claim the vehicle is linked to.

### Submission
Information about submissions grouped by subject.
- **Auto Liability:** A list of all submissions related to AL.
- **General Submissions:** A list of all general submissions.
- **Renewals:** A list of all Renewal submissions.
- **Endorsements:** A list of all Endorsement submissions.
- **Bind Requests:** A list of all Binds requested.

### Policies
Information about policies the user has access to.
- **All Policies:** A list of all policies.

### Claims
Information about Claims.
- **Loss Events:** A list of all loss events associated with a claim. Option to create a new loss event.
- **All Claims:** A list of all Claims the user has access to. Option to create a new loss event.
- **Claim Tasks:** A list of all Tasks associated with the available Claims.
- **Claim Reports:** Access to all reports related to Claims.
- **TPAs:** A list of Third Party Administrators. Restricted option, user may not have access.

### Quotes
Information about Quotes.
- **All Quotes:** A list of all Quotes the user has access to. Option to create a new Quote inside by pressing the button "+ New Business".
- **Quote Tasks:** A list of all Tasks associated with the available Quotes.
- **New Quote:** Option to create a new quote for insurance.

### Accounting
Information related to accounting.
- **Invoices:** A list of all Invoices the user has access to. Option to pay invoice by pressing the button "$ Pay Invoices".
- **Payments:** A list of all Payments made. Limited to what the user is authorized to see.
- **Bank Transactions:** A list of all bank transactions. Restricted option, user may not have access.
- **Reconciliation Workbench:** Bank transactions mapped to the ledger entries they've paid, across every realm — filter by Realm Type and Realm to narrow the view. Restricted option, user may not have access.
- **Tax Submission Queue:** Restricted option, user may not have access.

### Carriers
Information related to carriers.
- **Individual Carriers:** A list of all individual carriers the user has access to. Restricted option, user may not have access.
- **Carrier Groups:** A list of all carrier groups the user has access to. Restricted option, user may not have access.

### Agencies
Information related to agencies.
- **Manage Agencies:** A list of available agencies. Restricted option, user may not have access.
- **Reports:** A list of agency reports, such as commission reports.

### Premium Finance
Information related to premium financing. Limited for admin users.
- **Manage Premium Finance:** A list of premium finance companies.

### User Access
Access to users and user options. Access limited to Agency Administrators.
- **Users:** A list of all users accessible. Restricted option, user may not have access.
