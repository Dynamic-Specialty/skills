---
name: home-page-carrier-product-eligibility-lookup
description: >
  Use when answering a free-text home-page question that needs a carrier or product name
  resolved to its exact code, a carrier's eligibility criteria for a product looked up, or a
  cross-cutting eligibility question (a state, exclusion, or limit that might apply "for any
  carrier") answered by checking across multiple carriers/products rather than one named pair.
---

# Carrier, product, and eligibility lookup

## The one hard rule

Every carrier name, product name, abbreviation, and eligibility criterion in your answer must come
from a tool result received in this same turn. Never from memory. Never invented. Never inferred
from what a carrier or product "probably" is.

This covers every carrier and product your answer names, not only the one the user asked about.
If something is still missing or unclear after calling the tools, ask the user a direct question.

This skill has no scripts, references, or assets.

## Carrier names in your answer

Users are agencies. They don't know carrier abbreviated names.

- Always refer to a carrier by its full name. Never write its abbreviated name anywhere in the
  answer, not even in brackets next to the full name.
- `generate_carrier_criteria` and `list_carrier_products_available_options` return abbreviated
  names only. So whenever your answer names a carrier, also call `list_available_carriers` and
  swap each abbreviated name for its full name.
- This applies even if the user typed the abbreviated name themselves.
- Abbreviated names are only for passing to other tools.

## The four tools

| Tool | Returns | Is it an availability statement? |
|---|---|---|
| `list_available_carriers` | Each carrier's full name and its exact abbreviated name. | **Yes.** A carrier missing from this list is not available to the user. |
| `list_product_abbreviations` | Each active product's full name and its exact abbreviated code. | **No.** It is a glossary only. Never say a product is or isn't available based on it. |
| `list_carrier_products_available_options` | Every carrier/product combination the carrier offers and the user can access, grouped by carrier, with exact spellings. Some may have no eligibility rules at all. | **Yes**, for combinations. |
| `generate_carrier_criteria` | Eligibility criteria per carrier, for the carrier/product combinations you pass in. | Only for what it returns. See "Reading criteria results". |

Call rules:

- Call each of the three list tools **at most once per turn**. Reuse the result.
- Call `generate_carrier_criteria` **once per turn**, with every combination the question needs in
  that one call. Never one call per combination. A large batch is fine.
- `generate_carrier_criteria` needs the exact carrier abbreviated name and exact product code.
  Never guess either one. If unsure, resolve it with a list tool first.
- If a carrier the user asks about is missing from `list_available_carriers`, say it isn't
  available to them. Don't guess why. Don't suggest carriers outside that result.

## Pick the path the question needs

Use only the steps the question actually needs. There are three shapes.

1. **Name lookup only.** The user asks what an abbreviation means, or which carriers they can use.
   - Carrier by name, or "which carriers do we have": `list_available_carriers`.
   - Product by name, or "what does GL stand for": `list_product_abbreviations`.
2. **Named carrier and/or product, wants criteria.** Resolve any plain-language names first, then
   call `generate_carrier_criteria`. If you are not sure the combination exists for this user,
   check it against `list_carrier_products_available_options` first.
3. **Cross-cutting question.** No specific carrier/product is named. It asks about a state, an
   excluded commodity, a driver limit, or "is there any carrier that...". Follow
   "Cross-cutting questions" below. Never answer it from a single tool result.

## Reading criteria results

Criteria come back as free text (e.g. "Not available in AZ, CA, NV"). There are no structured
fields such as `state` to filter on. Read the text yourself.

When a combination comes back with no normal criteria, what it means depends on where the
combination came from:

| What came back | Where the combination came from | What it means | What to tell the user |
|---|---|---|---|
| Nothing | You typed or assumed it | The combination wasn't recognized: a spelling error, or it doesn't exist. | Confirm the carrier/product with the user. Don't say "nothing to report". |
| Nothing | `list_carrier_products_available_options` | Unclear. The carrier may simply have no rules for that product. | Say no criteria were found for it. Don't claim "excluded" or "no restrictions". |
| A single message saying the user lacks access | Either | An access restriction, not an eligibility rule. | Tell the user they don't have access to that carrier and product. Use the carrier's full name. Never present it as a criterion. |

## Cross-cutting questions

No tool answers these directly. Work them in three steps.

1. **List the combinations in scope.** Call `list_carrier_products_available_options`. It already
   returns only combinations that are offered and that this user can access.
   - Don't build the list by combining `list_available_carriers` with
     `list_product_abbreviations`. Those two aren't linked or filtered together, so you'd ask
     about combinations that don't exist or aren't visible to this user.
   - If the question names a product but no carrier, keep only that product's combinations. Same
     for a carrier with no product.
   - If the question scopes things in a way no tool knows about (e.g. "carriers we added this
     year"), say so. Don't guess.
2. **Get criteria for all of them in one call.** Pass every combination from step 1 to
   `generate_carrier_criteria`. Don't sample a subset. Don't trim the list to keep it small.
   Skipping combinations can produce a false "no carrier excludes it".
3. **Sort every combination into one of three groups** by reading its criteria text:
   - **Excluded**: the text says so.
   - **No exclusion found** in the returned text.
   - **Undetermined**: an empty result or an access message (see the table above).

   Answer by naming which carriers/products fall in each group. Don't collapse it to a bare
   yes/no.

## Glossary

Agencies use their own terms. Understand them in questions, and use them in your answers instead
of internal terms.

| Agency term | Meaning | Internal term |
|---|---|---|
| Appetite | What our underwriting guidelines are: what a carrier will and won't write. | Eligibility criteria |
| Underwriting guidelines | The rules a carrier applies before it will quote. | Eligibility criteria |
