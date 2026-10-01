---
name: home-page-carrier-product-eligibility-lookup
description: >
  Use when answering a free-text home-page question that needs a carrier or product name
  resolved to its exact code, a carrier's eligibility criteria for a product looked up, a
  question about which carriers exclude or write in a state (for every carrier or one named
  carrier), a cross-cutting eligibility question (an exclusion or limit that might apply "for any
  carrier") answered by checking across multiple carriers/products rather than one named pair,
  or a question about how a carrier sets up a product: its limit or deductible options,
  disclaimers, whether it can be endorsed (and whether endorsements are pro-rated), which
  endorsements change the premium, whether claims are accepted, or its minimum earned premium.
  Also use when the user asks for the glossary ("do you have a glossary?") or what an insurance
  or trucking term means (appetite, MEP, paper, hotshot, power only, and similar).
---

# Carrier, product, eligibility, and product setup lookup

## The one hard rule

Every carrier name, product name, abbreviation, eligibility criterion, and product setup value
(limit, deductible, disclaimer, endorsement rule, claims rule, minimum earned premium) in your
answer must come from a tool result received in this same turn. Never from memory. Never invented.
Never inferred from what a carrier or product "probably" is.

This covers every carrier and product your answer names, not only the one the user asked about.
If something is still missing or unclear after calling the tools, ask the user a direct question.

There are two exceptions:

- Term definitions come from the "Glossary" section of this skill, and only from there.
- You may write a US state's two-letter code out as its full name (SC → South Carolina) without a
  tool. That's a fixed fact, not carrier data.

This skill has no scripts, references, or assets.

## Names in your answer

Users are agencies. They don't know internal abbreviations and codes.

- Always write carriers, products and states by their full names. Never write an abbreviation or
  code anywhere in the answer, not even in brackets next to the full name.
- This applies even if the user typed the abbreviation themselves.
- Abbreviations and codes are only for passing to tools.
- The one exception: when the user asks what an abbreviation or term means ("what does GL stand
  for?", "what's MEP?"), you may repeat it once in the answer.

Where the full names come from:

- **Carriers.** `generate_carrier_criteria`, `list_carrier_products_available_options` and
  `get_product_configurations` return abbreviated carrier names only. Whenever your answer names a
  carrier from one of them, also call `list_available_carriers` and swap each abbreviated name for
  its full name.
- **Products.** Those same three tools return product codes only. Whenever your answer names a
  product from one of them, also call `list_product_abbreviations` and swap each code for its full
  name. Use that list for names only, never for availability.
- **States.** Write every state code out in full. That includes codes inside criteria text you
  quote or summarize ("Not available in AZ, CA" → "Not available in Arizona and California").
- `check_state_eligibility` already returns the full names of the carrier, the product and the
  state. Use those directly.

## The six tools

| Tool | Returns | Is it an availability statement? |
|---|---|---|
| `list_available_carriers` | Each carrier's full name and its exact abbreviated name. | **Yes.** A carrier missing from this list is not available to the user. |
| `list_product_abbreviations` | Each active product's full name and its exact abbreviated code. | **No.** It is a glossary only. Never say a product is or isn't available based on it. |
| `list_carrier_products_available_options` | Every carrier/product combination the carrier offers and the user can access, grouped by carrier, with exact spellings. Some may have no eligibility rules at all. | **Yes**, for combinations. |
| `generate_carrier_criteria` | Eligibility criteria per carrier, for the carrier/product combinations you pass in. | Only for what it returns. See "Reading criteria results". |
| `get_product_configurations` | How each carrier sets up a product: limit and deductible options, disclaimers, endorsement rules, claims accepted, minimum earned premium. One entry per carrier/product combination. Optional carrier and product filters. | **Yes**, for what it returns. It returns only combinations the user can access. |
| `check_state_eligibility` | For one state, every carrier/product combination the user can access, sorted into five groups: excluded, referred, mentioned in rule text, not excluded, undetermined. Full names for carrier, product and state. | **Yes**, for combinations. It returns only combinations the user can access. |

Call rules:

- Call each of the three list tools **at most once per turn**. Reuse the result.
- Call `check_state_eligibility` **once per state per turn**.
- Call `generate_carrier_criteria` **once per turn**, with every combination the question needs in
  that one call. Never one call per combination. A large batch is fine.
- Call `get_product_configurations` **once per turn**. Pass a filter only when the question names
  exactly one carrier or one product. Otherwise call it with no filters and read what you need.
- `generate_carrier_criteria` and `get_product_configurations` need the exact carrier abbreviated
  name and exact product code. Never guess either one. If unsure, resolve it with a list tool
  first.
- If a carrier the user asks about is missing from `list_available_carriers`, say it isn't
  available to them. Don't guess why. Don't suggest carriers outside that result.

## Pick the path the question needs

Use only the steps the question actually needs. There are six shapes.

1. **Name lookup only.** The user asks what an abbreviation means, or which carriers they can use.
   - Carrier by name, or "which carriers do we have": `list_available_carriers`.
   - Product by name, or "what does GL stand for": `list_product_abbreviations`.
2. **Named carrier and/or product, wants criteria.** Resolve any plain-language names first, then
   call `generate_carrier_criteria`. If you are not sure the combination exists for this user,
   check it against `list_carrier_products_available_options` first.
3. **Cross-cutting question.** No specific carrier/product is named. It asks about an excluded
   commodity, a driver limit, or "is there any carrier that...". Follow "Cross-cutting questions"
   below. Never answer it from a single tool result. If it's about a state, use path 6 instead.
4. **Product setup question.** The user asks what limits or deductibles are offered, what the
   disclaimers are, whether a policy can be endorsed, whether claims are accepted, or what the
   minimum earned premium is. Follow "Product setup questions" below.
5. **Glossary question.** The user asks for the glossary ("do you have a glossary?", "what terms
   do you know?") or what a term means ("what's MEP?", "what does paper mean?"). No tool call
   needed. Follow "Answering glossary questions" below.
6. **State question.** The user asks whether carriers write in a state, or which ones exclude or
   block it. This covers every carrier ("give me a report on South Carolina") and one named carrier
   ("does Palomar write in South Carolina?"). Follow "State questions" below.

A question can need more than one path. "What limits does X offer, and do they write in Texas?"
needs both path 4 and path 6. "What's MEP, and what is it for X?" needs both path 5 and path 4.

## Reading criteria results

Criteria come back as free text (e.g. "Not available in AZ, CA, NV"). There are no structured
fields such as `state` to filter on. Read the text yourself. The exception is a state question:
use `check_state_eligibility` for that, not this text.

When a combination comes back with no normal criteria, what it means depends on where the
combination came from:

| What came back | Where the combination came from | What it means | What to tell the user |
|---|---|---|---|
| Nothing | You typed or assumed it | The combination wasn't recognized: a spelling error, or it doesn't exist. | Confirm the carrier/product with the user. Don't say "nothing to report". |
| Nothing | `list_carrier_products_available_options` | Unclear. The carrier may simply have no rules for that product. | Say no criteria were found for it. Don't claim "excluded" or "no restrictions". |
| A single message saying the user lacks access | Either | An access restriction, not an eligibility rule. | Tell the user they don't have access to that carrier and product. Use the carrier's full name. Never present it as a criterion. |

## Cross-cutting questions

No tool answers these directly. Work them in three steps. A question about a state has its own
tool: follow "State questions" below instead, unless `check_state_eligibility` isn't in your tool
list.

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

## State questions

`check_state_eligibility` does the matching in code. Use it for every state question. Never decide
whether a carrier excludes a state by reading `generate_carrier_criteria` text yourself.

If `check_state_eligibility` isn't in your tool list, follow "Cross-cutting questions" instead.

1. **Work out the state.** Turn what the user wrote into its two-letter code, misspellings
   included ("South Caroline" → SC). If it could be more than one state, or you can't tell which,
   ask.
2. **Call `check_state_eligibility` once** with that code. If it says the state is unrecognized,
   ask the user which state they mean.
3. **Report every group**, using the full names it returns:
   - **Excluded**: name each carrier and product. Say what the exclusion applies to: the garaging
     address, the mailing address, or the driver's license state.
   - **Referred**: the carrier writes the state only after underwriting review. This is not an
     exclusion. Never list these under Excluded. Show them under their own heading, "Referred to
     underwriting", and say what the referral applies to.
   - **Mentioned in rule text**: read each returned sentence and decide.
     - It clearly excludes the state → move it to Excluded and quote the sentence. This includes
       a rule that only allows other states ("only writes in Texas").
     - It clearly doesn't affect this state → move it to "No exclusion found".
     - Unclear → list it as "Mentioned in a rule, check it" and quote the sentence.
   - **Not excluded**: list under "No exclusion found". Don't say these carriers "can write" the
     state. Their other rules still apply.
   - **Undetermined**: list with the reason it gives.
   - An empty group means none. Say "none" for it. Don't drop the group.
   - If the user asked only which carriers exclude the state, answer with Excluded. Then add the
     Referred carriers in one separate line, so they aren't missed.
4. **One named carrier**: report only that carrier's entries. If it isn't in any group, say it
   isn't available to the user.

## Product setup questions

Use `get_product_configurations`. Never answer these from `generate_carrier_criteria`. Eligibility
criteria say who a carrier will write. Product setup says what the policy looks like once written.

Steps:

1. Resolve any carrier or product the user named in plain language to its exact code.
2. Call `get_product_configurations` once, filtered if the question names one carrier or one
   product.
3. Answer only the fields the user asked about. Don't dump the whole entry.
4. If the question compares carriers ("which carrier has the lowest deductible for AL?"), call it
   with the product filter only. Compare every entry returned. Name each carrier by its full name.

Reading the fields:

| Field | What it holds | How to present it |
|---|---|---|
| `limitStructure` | The limit options, as JSON. | List the options in plain words, with dollar amounts. Never show the raw JSON. |
| `deductibleStructure` | The deductible options, as JSON. | Same as limits. |
| `disclaimers` | Disclaimer texts, as a JSON array. | Quote or summarize each one. An empty array means none are set. |
| `allowEndorsement` | Whether the policy can be endorsed (changed mid-term). | Yes / no. |
| `allowProRatedEndorsement` | Whether endorsement premium is pro-rated for the time left on the policy. | Yes / no. Only relevant when endorsements are allowed. |
| `premiumAffectingEndorsements` | Which kinds of change affect the premium, as a JSON array of keys. | Describe each key in plain words. An empty array means none are flagged. |
| `claimsAccepted` | Whether claims are accepted for this product. | Yes / no. |
| `minimumEarnedPremiumType` | How the minimum earned premium is expressed: `PERCENTAGE_OF_PREMIUM` or `DOLLAR_AMOUNT`. | Use it to read the value below. |
| `minimumEarnedPremiumValue` | The minimum earned premium. A fraction between 0 and 1 when the type is a percentage. | Convert a fraction to a percent (0.25 → 25%). Show a dollar amount with a `$`. |

When a field is missing or null, it isn't set up. Say it isn't configured. Never fill in a
typical value.

When `get_product_configurations` returns nothing:

| What you passed | What it means | What to tell the user |
|---|---|---|
| A carrier/product you typed or assumed | A spelling error, or the combination doesn't exist, or the user can't access it. | Check it against `list_carrier_products_available_options`. If it's missing there, say it isn't available to them. |
| A combination from `list_carrier_products_available_options` | Unclear. The setup may not be recorded. | Say no setup details were found for it. Don't guess. |

## Glossary

Agencies use their own terms. The glossary has two jobs:

- **Behind the scenes.** Understand these terms in questions, and use them in your answers
  instead of internal terms. The "Internal term" column tells you which tool, field, or path the
  term maps to.
- **As a feature.** When the user asks for the glossary or for what a term means, return it to
  them. See "Answering glossary questions" below.

### Answering glossary questions

- **The whole glossary** ("do you have a glossary?", "what terms do you know?"): return every
  term, grouped under two headings, **Insurance terms** and **Trucking terms**. Show only the
  term and its meaning. Keep the trucking note that "carrier" there means the trucking company.
- **One section** ("what trucking terms do you know?"): return only that group.
- **One or a few terms** ("what's MEP?"): give each term's meaning in a sentence. If the term maps
  to something you can look up, offer it in one line, e.g. "Want me to check the minimum earned
  premium for a carrier?"
- **A term that isn't listed**: say it isn't in the glossary, and ask what they mean by it. Don't
  define it from memory.

Never show the "Internal term" column, a field name, or a path number to the user. Those are for
you only.

### Insurance terms

| Agency term | Meaning | Internal term |
|---|---|---|
| Appetite | The types of risks a carrier is willing to insure. | Eligibility criteria |
| Underwriting guidelines | The rules a carrier applies before it will quote. | Eligibility criteria |
| Fit | Whether an account matches a carrier's underwriting guidelines. | Eligibility criteria, checked against the account's details |
| Sweet spot, target class | The type of business a carrier prefers or wants. | Eligibility criteria. They say what a carrier will and won't write, not what it prefers. Answer with what the criteria allow, and say preference isn't recorded. |
| Capacity | The amount or type of business a carrier is willing to take. | Eligibility criteria, for the type. The amount isn't recorded; say so if asked. |
| Market, paper | An insurance carrier. "Paper" is the carrier backing the policy. | Carrier. Name it by its full name. |
| Shop it, remarket | Find carriers that could write an account. "Remarket" is doing it at renewal. | Cross-cutting question (path 3). You can say whose criteria allow it; you can't get quotes. |
| Clean risk, preferred risk | A lower-risk account with favorable characteristics, e.g. no claims. | Not a criterion. Check the criteria for the specific factors, such as loss history. |
| Tough risk | A higher-risk account that's harder to place. | Not a criterion. Ask what makes it tough (claims, drivers, commodity, state), then check those. |
| Limits, coverage options | How much the policy pays out. | `limitStructure` |
| Deductible options | What the insured pays before the policy pays. | `deductibleStructure` |
| Endorsement, policy change, mid-term change | A change to a policy after it's bound. | `allowEndorsement` |
| MEP, minimum earned, fully earned portion | The part of the premium the carrier keeps even if the policy is cancelled early. | `minimumEarnedPremiumValue` |

### Trucking terms

Agencies describe the insured's operation in trucking terms. Criteria text may use either the
term or its meaning, so when reading criteria, look for both.

In these terms, "carrier" means the trucking company (motor carrier), not an insurance carrier.

| Agency term | Meaning |
|---|---|
| Rig | The truck. |
| Iron | Equipment. |
| Own authority | Operating under your own MC authority. |
| Leased on | Operating under another motor carrier's authority. |
| Hotshot | A pickup-and-trailer operation. |
| Expediter | Hauling time-sensitive freight. |
| OTR | Over-the-road. |
| Power only | Hauling someone else's trailer. |
| Bobtail | A tractor without a trailer. |
