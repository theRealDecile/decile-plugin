---
name: us-population-comparison
description: "Benchmark a customer population's demographics and interests against US population distributions using Decile, showing how much more or less likely these customers are to have a given attribute than the average American. Use whenever the user asks how their customers compare to the US population, the general market, or the average American; which attributes over- or under-index; what their customers are into; or wants any interest, hobby, or psychographic affinity breakdown — including \"who are our customers really\", \"what's distinctive about our buyers\", \"are we over-indexed on high earners\", \"do we skew older than the market\", or \"what should we know for ad targeting\". Also trigger for market-representation questions and audience descriptions that compare against outside population data. Covers demographic and interest enrichment attributes only; defaults to all purchasing customers when no population is named."
---

# US Population Comparison

## When to use this skill

Reach for this skill when the question is about how a customer population compares to the **US population** — the outside market — rather than how it compares to the brand's own customers. Signals this is the right skill:

- An explicit market comparison ("vs. the average American," "vs. the US population," "vs. the general market")
- A question about over- or under-indexing on any attribute
- Any interest, hobby, or psychographic affinity question ("what are our customers into," "what do they care about outside our category")
- Market-representation framing: ad targeting, creative direction, partnership or sponsorship selection, positioning
- A request to characterize who the customers are in demographic or lifestyle terms, with no behavioral or revenue component

This skill works entirely in Decile's enrichment layer — demographic and interest attributes, benchmarked against US population data. It does not cover revenue, order, retention, or product-mix metrics. If the user wants those alongside the benchmark work, cover the benchmark half here and say plainly which half you covered.

If the user asks for a single specific distribution with no benchmark ("what percent of our customers are women?"), just answer it — this skill is for the fuller comparison.

## Workflow

1. **Establish brand context.** Call `get_default_brand` before querying anything. If the user names a brand that isn't the default, use `list_brands` with `name_search` to find it. Pass `brand_ref` to every `query_data` call exactly as returned — never construct one. Don't combine brands in a single profile; comparisons from two brands read as equivalent when they aren't.

2. **Scope the population.** Default to all purchasing customers. Otherwise extract the population from the request — a product's buyers, an acquisition channel, a subscription status, an LTV tier, a geography, an acquisition cohort, or a saved Decile audience (include the audience id in the query text). State the definition you used in one line so the user can correct a wrong assumption. If scope is ambiguous, proceed with all customers and note it rather than blocking.

3. **Query one attribute at a time.** Decile benchmarks a single attribute per query, so issue one `query_data` call per attribute and send them in the same turn to run in parallel. A single call covering every attribute at once will fail or return something unreliable. Each call needs `brand_ref`, `original_user_prompt` (the user's most recent message copied verbatim, unedited), and a precise `question`. Pass relative time references ("last 90 days") through verbatim rather than converting to dates.

   Per-attribute template:
   ```
   For [POPULATION], show the distribution of [ATTRIBUTE] as a percent of customers, alongside
   the US population benchmark and the index value for each bucket (100 = same as US average).
   Include the total customer count and the enrichment coverage for this attribute.
   ```

   Interests take a single call, not one per interest:
   ```
   For [POPULATION], return the top 25 over-indexed and top 10 under-indexed interest categories
   ranked by US-population index value. For each, include the customer share, the US benchmark
   share, and the index.
   ```

   Check the interest result before trusting it: if nothing clears parity, the denominators don't match and the ranking is meaningless. See Reporting the comparison.

4. **Check coverage before interpreting.** These comparisons are ratios, and ratios on thin data produce confident-looking nonsense. Apply the guardrails in Reporting the comparison below before writing anything.

5. **Build the output** following the exact structure in Output structure.

## Analysis coverage

Cover six to eight attributes plus interests. More dilutes the narrative; fewer risks missing the defining divergence. If the user names specific attributes, cover those and skip the rest.

- **Core demographics** — age group, gender, household income, education level, presence of children, home ownership
- **Extended demographics**, when the request or an early finding warrants it — net worth, marital status, population density, occupation, geography
- **Interests / psychographics** — the strongest over- and under-indexed categories

Rank interests by index, never by raw count or customer-base share. A category can be the most common thing among customers while being no more common than in the general population, which makes it useless for targeting.

## Reporting the comparison

Decile returns index values, where 100 equals the US average. **Never surface a raw index or a bare decimal.** "Index 216" means close to nothing to a reader; "2.2x the rate of the average American" lands immediately. Convert every comparison before it reaches the output — chart labels, narrative, and summary cards alike.

Two scales come back, so normalize before converting. Demographic queries return the index on a 100 scale — `154` means 1.5x. Interest queries return a raw ratio — `1.54` means 1.5x. Mixing them up turns a 0.23 ratio into "index 23" or a 154 index into "154x."

| Decile returns | Present as |
|---|---|
| 216 | 2.2x as likely — or "more than twice as likely" |
| 135 | 35% more likely |
| 100 | in line with the US average |
| 65 | 35% less likely |
| 30 | about a third as likely |

Multipliers read better past roughly 1.5x; percentages read better near parity. Phrase multipliers as "as likely" or "the rate of," and reserve "more likely" for the percentage form — "2.2x more likely" strictly means 3.2x the rate, which is not what the data says. Round to one decimal at most.

Guardrails, in the order they bite:

| Check | Threshold | What to do |
|---|---|---|
| Enrichment coverage | Below ~50% of the population | Caveat the whole profile up front — it describes the enriched subset, not all customers. Coverage varies by attribute, so report the range rather than a single figure |
| Bucket size | Under ~5% of the customer base being profiled | Don't rank it as a headline finding; report as directional or omit |
| Interest denominator | No interest category clears parity | The customer share was computed over all purchasers while the benchmark side is fully enriched, so every category under-indexes by construction. That's an artifact, not a finding — say so instead of ranking it |
| Correlated attributes | Age, income, net worth, children, home ownership move together | Collapse findings that are all downstream of one fact into one finding |
| Empty or errored result | — | Say so plainly and suggest a broader population; don't fill the gap with estimates |

A 4x skew on 1% of customers is noise wearing a confident number. Always pair the comparison with the underlying share so magnitude stays visible.

## Output structure

Always structure the final response as:

1. **Comparison bar chart** — see Visualization below.
2. **What's most different** — 3–5 items ranked by how much they should change someone's mental model of this customer, not strictly by the size of the gap. For each: the customer share, the US share, and what it means in plain terms. Lead with the finding, not the metric.
   > **Children in the household:** 67% of customers are parents vs. 31% nationally — 2.2x the rate of the average American. This is bought by people managing a household, not just themselves.
3. **Psychographic themes** — group interests into 2–4 named themes rather than listing categories one by one; a flat list of 25 interests is data, not insight. Give each theme its 2–3 strongest signals, stated as multipliers. Then call out counter-intuitive under-indexes, where an obvious assumption about this customer turns out to be wrong — often the most useful line in the output, because it corrects an assumption someone was about to act on.
4. **Core archetype** — one concrete sentence capturing who this customer is. "Suburban 35–44 parents with household income above $100k who cook, travel with kids, and follow home improvement" beats "health-conscious modern families."

If a non-default population was profiled, add one line on how it differs from the overall base.

**When the data fails the guardrails, don't run this structure anyway.** Lead with what's unreliable and why, report only the findings that clear the thresholds, and cut the chart down to the buckets that survived — or drop it. A chart implies the numbers can bear weight, so rendering one over thin coverage does more damage than returning less.

### Visualization

Horizontal bar chart, demographics and interests stacked vertically in one widget (not tabbed — hidden sections stream invisibly). Bars scaled to the largest divergence in each group and labelled with the multiplier and the raw customer share — the multiplier alone hides magnitude. Include a reference line or legend marking parity with the US average, distinguish over- from under-index visually, and flag low-confidence buckets rather than dropping them silently.

Lead with metric cards for total customers, enrichment coverage, and the three strongest divergences.

## Notes

- Enrichment attributes are for aggregate analysis only. Never attach a demographic or interest value to an individual customer or CUSTOMER_ID.
- Don't call this output a "persona." Personas is a named Decile feature — use it only in reference to that feature. Say profile, archetype, or audience description instead.
- Decile data runs on a 24-hour delay; today's activity isn't included.
- These comparisons are directional. They show over-representation against a benchmark — not causation, and not which attributes drive revenue.
