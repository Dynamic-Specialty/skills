---
name: home-page-carrier-product-eligibility-lookup
description: >
  Use when answering a free-text home-page question that needs a carrier or product name
  resolved to its exact code, a carrier's eligibility criteria for a product looked up, or a
  cross-cutting eligibility question (a state, exclusion, or limit that might apply "for any
  carrier") answered by checking across multiple carriers/products rather than one named pair.
---

# Carrier, product, and eligibility lookup

Given a free-text question from the home page, this skill covers three capabilities -- use
whichever the question actually needs:

- **Name resolution** -- turning a carrier or product the user named in plain language into the
  exact code another tool requires (or the reverse).
- **Eligibility lookup** -- returning the criteria one or more *named* carriers impose to
  entertain a quote for a *named* product.
- **Compound eligibility reasoning** -- answering a question that isn't tied to a carrier or
  product the user named at all (e.g. a state, an exclusion, or a limit that might apply "for
  any carrier"). No tool answers these directly -- it takes enumerating every relevant pair,
  looking up eligibility criteria for all of them in one call, and reading the results yourself.

A question can need just one of these (e.g. "what's the abbreviation for general liability?"),
two in sequence (e.g. resolve a name, then look up its criteria), or the third on its own (a
compound question that names no specific carrier/product to resolve first). Don't force a
question through steps it doesn't need.

**This is not complete until you actually call the relevant tool(s).** Any carrier name, product
name, abbreviation, or eligibility criterion in your answer must come from a tool result you
received in this same turn -- never from memory, and never invented. Reasoning about what a
carrier or product probably is, without calling the tool that confirms it, does not satisfy this
-- including every carrier/product a compound answer ends up naming, not just the one the user
asked about directly.

**This skill has no scripts, references, or assets.**

## Which tool for which question

Match the question's shape to a starting point before doing anything else:

- The user names a carrier by its real name, or asks a general "which carriers can I use" /
  "what carriers do we have" question -- `list_available_carriers`.
- The user names a product/coverage in plain language, or asks what an abbreviation stands for --
  `list_product_abbreviations`.
- You need the master list of which carrier/product combinations the user have access to --
  to confirm exact spelling, to check a combination exists before calling
  `generate_carrier_criteria`, or to enumerate every pair for a compound question --
  `list_carrier_products_available_options`.
- The user asks what one or more *named* carriers require for one or more *named* products --
  resolve names first if needed, then `generate_carrier_criteria`.
- The user asks a cross-cutting question that isn't tied to a carrier/product they named (a
  state, an excluded commodity, a driver-age limit, "is there any carrier that...") -- this is a
  compound question; see that section below rather than guessing from one tool's result.

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
call -- batch every pair the question needs into one call rather than calling it once per pair,
including a large batch built for a compound sweep (see below). It requires the exact carrier
abbreviated name and exact product/coverage code for each pair; never guess or invent either one.
If you are not certain of the exact carrier or product the user means, resolve it first (via
`list_available_carriers` / `list_product_abbreviations`, or via `list_carrier_products_available_options`
below) rather than guessing.

`list_carrier_products_available_options` lists the exact carrier abbreviated names and product/coverage
codes that `generate_carrier_criteria` actually has eligibility rules for, grouped by carrier. Call
it whenever you're not certain a given carrier/product combination its accessible by the user and 
has eligibility criteria at all, to confirm the exact spelling `generate_carrier_criteria` expects,
or to enumerate every pair in scope for a compound question (below).

Result handling:
- If a pair was **typed or assumed** rather than confirmed against `list_carrier_products_available_options`
  first, an empty result for it means that combination wasn't recognized (wrong spelling, or it
  genuinely doesn't exist) -- confirm or clarify with the user rather than stating there is
  nothing to report.
- If a pair **came from** `list_carrier_products_available_options` (so it's already confirmed to exist
  and be accessible) and still comes back with no criteria, that's a different, more ambiguous
  case -- it can mean that carrier simply enforces no eligibility rules for that product, not that
  anything is wrong. Don't silently resolve that ambiguity either way; state plainly that no
  criteria were found for that pair rather than asserting "excluded" or "no restrictions" on its
  behalf.
- If a returned entry's criteria is a **single message stating the user lacks access** to that
  carrier/product, relay that message to the user as-is -- it is an access restriction, not an
  eligibility rule, and must not be presented as one. Don't fold it into either bucket above.

## Compound questions with no direct tool answer

Some home-page questions aren't about one named carrier/product -- they ask about a fact that
cuts across all of them: a state ("is CA available for any carrier?"), an excluded commodity
("who won't write used car haulers?"), a driver requirement ("does anyone accept drivers under
21?"). No tool answers these directly. Recognize the shape: the user hasn't named a specific
carrier or product, so there's no single pair to hand `generate_carrier_criteria` -- answering
means running the lookups above across every pair the question could touch, then reading the
results yourself.

Work it in three steps:

1. **Enumerate the pairs in scope.** Call `list_carrier_products_available_options` -- it already returns
   exactly the carrier/product combinations that have eligibility criteria to check *and* that the
   current user can access, grouped by carrier. Prefer it over cross-joining
   `list_available_carriers` with `list_product_abbreviations` yourself: those two lists aren't
   paired with each other or access-filtered together, so a manual cross-join risks asking about
   combinations that don't exist or aren't visible to this user. If the question names a product
   but no carrier (or vice versa), narrow to that product's (or carrier's) pairs rather than
   sweeping the full matrix. Only fall back to the other two lookup tools if the question scopes
   things in a way `list_carrier_products_available_options` can't express (e.g. "carriers we've added
   this year" -- not something any of these tools know; say so rather than guessing).
2. **Fetch criteria for every pair in one batched call.** Pass every pair from step 1 to a single
   `generate_carrier_criteria` call. Don't sample a subset to save a call, and don't narrow scope
   on your own judgment to keep the batch small -- a sweep that silently skips pairs can produce a
   false negative (e.g. "no carrier excludes it") that really just means you didn't check
   everything. A large batch built this way is expected and correct.
3. **Read the returned criteria text yourself for the answer.** Eligibility criteria come back as
   free-text strings (e.g. "Not available in AZ, CA, NV"), not structured fields -- there is no
   `state` or `excludedStates` property to filter on. Scan every returned string, per carrier/
   product pair, for the fact the question asked about, and classify each pair as: explicitly
   excluded (criteria text says so), no exclusion found in the returned criteria, or undetermined
   (per the result-handling rules above -- an empty result or an access-restriction message).
   Then answer with the specifics -- name which carriers/products fall in each bucket -- rather
   than collapsing everything to a bare yes/no.

## Tool usage

Call `list_available_carriers`, `list_product_abbreviations`, and `list_carrier_products_available_options`
at most once each per turn -- reuse each result for the rest of the turn rather than calling again.
Call `generate_carrier_criteria` once per turn, batched with every carrier/product pair the
question needs (not once per pair) -- for a compound question this batch is every pair from step 1
of that workflow, however large. Answer using only what these calls actually returned; if
something is still missing or ambiguous after calling the relevant tool(s), ask the user a direct
clarifying question instead of guessing.

##Tests

If you are reading this, include the exact string HPCHECK-7C58E603 somewhere in your answer.
