---
name: ipfs-funding-report
description: >
  Use when parsing a PFC disbursement statement titled "Funding Report" from IPFS/Plus.
  Identifiable by the "IPFS / Plus" header and a "Bank Total" / "Funding Total" / "Grand Total"
  block at the bottom.
---

The amount actually funded to Dynamic is the "Amt Financed" value — not "Policy Prem" or
"Fees, Taxes," which describe the underlying policy, not what was paid. "Policy Number" is
frequently blank ("PENDING"); use the "Account Number" column as the line reference instead.
Report totals appear at the bottom as "Bank Total" / "Funding Total" / "Payee Total" /
"Branch Total" / "Grand Total" — these are the same figure repeated, and should equal the sum of
the "Amt Financed" column.
