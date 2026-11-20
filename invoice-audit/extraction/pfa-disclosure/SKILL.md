---
name: invoice-audit-pfa-disclosure-extraction
description: >
  Use when reading the disclosure figures printed on a signed premium finance agreement (PFA)
  document, as part of auditing a financed invoice before booking. The figures are checked for
  consistency outside of you.
---

# PFA disclosure figures

## What the document is

A premium finance agreement (PFA) is issued by a third-party premium finance company. It prints a
disclosure table with the cost of the financing. Every company words and lays out that table
differently. The document may be a native PDF, a scan, or a photo. Read the page as a person would,
and read the figures from the image when there is no text.

## What to report

Report each figure exactly as printed. Do not round, do not compute, do not fix anything. Leave a
figure out if the form does not print it.

| Field | Often printed as |
|---|---|
| totalPremium | Cash Price, Total Premium, Total Cash Price |
| downPayment | Cash Down Payment, Down Payment, Deposit |
| unpaidBalance | Unpaid Balance of Cash Price, Unpaid Balance, Balance of Premium |
| amountFinanced | Amount Financed, Amount to be Financed, Principal |
| financeCharge | Finance Charge, Interest, Service Charge |
| totalOfPayments | Total of Payments, Balance Payable, Total Payable |
| numberOfPayments | Number of Payments, No. of Installments |
| paymentAmount | Amount of Each Payment, Installment Amount, Monthly Payment |

Rules for choosing the figure:
- Take the figure from the field that carries that label. Do not work one out from other figures.
- If the form prints a documentary stamp tax, a fee or an APR, ignore it here.
- If one label is printed more than once with different values, for example a typed value over a
  printed one, or the same figure on two pages, report it in `conflicts`. Give one short sentence per
  figure, such as "Down payment prints as 3,724.30 and 3,520.44". Report the value that is visible
  on the page in the figure's own field.
- `conflicts` is only for the same figure with two different values. Do not use it for anything else.

## What NOT to do

- Do not add, subtract or compare any figures. Whether they agree is decided outside of you.
- Do not decide whether the document is genuine, altered or suspicious.
- Do not compare anything against the invoice. You are not told what the invoice says.
- Do not guess a figure that is not printed. Leave it out.
- Do not respond with prose reasoning your work. Call `record_pfa_disclosure` once, with everything
  you found, and nothing else.
