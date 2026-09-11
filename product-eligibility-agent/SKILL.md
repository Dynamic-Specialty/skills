---
name: home-page-carrier-product-eligibility-lookup
description: >
  Use when answering a free-text home-page question that needs a carrier or product name
  resolved to its exact code, and/or a carrier's eligibility criteria for a product looked up.
---

# Carrier, product, and eligibility lookup

Given a free-text question from the home page, this skill covers two genuinely separate
capabilities -- use whichever the question actually needs, either or both:

- **Name resolution** -- turning a carrier or product the user named in plain language into the
  exact code another tool requires (or the reverse).
- **Eligibility lookup** -- returning the criteria one or more carriers impose to entertain a
  quote for a product.

A question can need just one of these (e.g. "what's the abbreviation for general liability?"),
just the other (e.g. "what does ASIC require for general liability?", where the user already used
exact names), or both in sequence (resolve first, then look up). Don't force a question through
both steps if it doesn't need them.

**This is not complete until you actually call the relevant tool(s).** Any carrier name, product
name, abbreviation, or eligibility criterion in your answer must come from a tool result you
received in this same turn -- never from memory, and never invented. Reasoning about what a
carrier or product probably is, without calling the tool that confirms it, does not satisfy this.

**This skill has no scripts, references, or assets.**

## Resolving carrier names

`list_available_carriers` lists every carrier the current user can see, full name alongside its
exact abbreviated name. Call it whenever the user refers to a carrier by its real name, to resolve
the exact abbreviated name other tools require, or to answer a general "which carriers can I use"
question.

Unlike the product list below, **this result IS an availability statement.** Only treat a carrier
as available to the user if `list_available_carriers` actually returned it. If a carrier the user
asks about is missing from that list, tell them it isn't available to them rather than guessing
why or naming carriers outside that result.

## Resolving product names

`list_product_abbreviations` returns a reference list of every active product's name and exact
abbreviated code. **This result is background terminology only, not an availability statement** --
the opposite framing from the carrier list above. Never tell a user a product is or isn't
available to them based on this list alone; that is determined by whichever tool actually performs
the action (e.g. `generate_carrier_criteria`'s own result, per below). Use it only to resolve a
product the user named in plain language (e.g. "general liability") into the exact abbreviated
code another tool requires, or vice versa.

Never guess or invent a carrier or product abbreviation that neither list actually returned.

## Eligibility criteria lookup

`generate_carrier_criteria` returns the eligibility criteria one or more carriers impose to
entertain a quote, organized by carrier. It accepts multiple carrier/product pairs in a single
call -- batch every pair the question needs into one call rather than calling it once per pair.
It requires the exact carrier abbreviated name and exact product/coverage code for each pair;
never guess or invent either one. If you are not certain of the exact carrier or product the user
means, resolve it first (via `list_available_carriers` / `list_product_abbreviations`, or via
`list_carrier_eligibility_options` below) rather than guessing.

`list_carrier_eligibility_options` lists the exact carrier abbreviated names and product/coverage
codes that `generate_carrier_criteria` actually has eligibility rules for, grouped by carrier. Call
it whenever you're not certain a given carrier/product combination has eligibility criteria at all,
or to confirm the exact spelling `generate_carrier_criteria` expects.

Result handling:
- An **empty** result from `generate_carrier_criteria` for a requested pair means that carrier/
  product combination was not recognized, not that there are no criteria -- confirm or clarify
  with the user rather than stating there is nothing to report.
- If a returned entry's criteria is a **single message stating the user lacks access** to that
  carrier/product, relay that message to the user as-is -- it is an access restriction, not an
  eligibility rule, and must not be presented as one.

## The category parameter

`generate_carrier_criteria` accepts an optional `category` per carrier/product pair, to filter
criteria to one of exactly four values: `unit`, `GENERALFREIGHT`, `commodity`, or `driver` (this
list -- the field's own documented description -- is the authoritative source; do not trust any
other wording you may see elsewhere).

Only pass a category when the user's own words clearly specify one of these four concepts (e.g.
they ask specifically about driver requirements, or about a commodity/freight type). Never map a
vague or unrelated word onto one of these four, and never invent a fifth value. When in doubt,
leave it blank -- an unfiltered result is always safe; an over-filtered one can silently hide
criteria the user actually asked about.

## Tool usage

Call `list_available_carriers`, `list_product_abbreviations`, and `list_carrier_eligibility_options`
at most once each per turn -- reuse each result for the rest of the turn rather than calling again.
Call `generate_carrier_criteria` once per turn, batched with every carrier/product pair the
question needs (not once per pair). Answer using only what these calls actually returned; if
something is still missing or ambiguous after calling the relevant tool(s), ask the user a direct
clarifying question instead of guessing.
