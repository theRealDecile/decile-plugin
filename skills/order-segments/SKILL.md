---
name: order-segments
description: "Define and analyze order segments using Decile — groups of transactions rather than groups of people. Use whenever the unit being counted is an order: \"what share of our orders used a discount\", \"how much revenue comes from subscription vs one-time orders\", \"compare weekend and weekday orders\", \"what do first orders look like vs repeat orders\", \"profile our winback orders\", \"what's the return rate on bundle orders\", \"retail vs online mix\", \"are multi-product orders worth more\". Also use to translate a plain-English description of a transaction type (\"orders where someone got something for free\", \"orders that came back to us\") into concrete order-level filters, and to decide which orders to exclude as non-revenue noise before computing any order metric. If the cohort is defined by an order property but the answer is counted per person — \"are discount-acquired customers worth less\", \"do subscription customers order more often\" — supply the order-level filter here and hand the analysis to segment-profiles."
---

# Order Segments

## When to use this skill

Ask what a single row in the answer represents. A transaction is this skill; a person is
`segment-profiles`.

- "What share of orders used a discount?" → orders → this skill
- "What share of customers usually buy on discount?" → people → `segment-profiles`

That test only resolves questions where the filter and the unit agree. When a cohort is
*defined* by an order property but *measured* per person — "are customers who first bought
on a discount worth less," "do subscription customers order more often" — the answer is per
person, so `segment-profiles` owns it. Define the order-level filter here, state it
explicitly, and hand it over. Don't split the write-up across two answers; one skill
produces the response, and the other contributes a definition.

That hand-off still applies when the measure is a lifetime aggregate, even though the
question sounds longitudinal. "Are discount-acquired customers worth less" is answerable as
a point-in-time comparison of two cohorts on lifetime value or repeat rate — it needs a
caveat that recently acquired customers haven't had time to repeat yet, not a refusal.

What is outside both skills is analysis indexed *by* time: retention curves, time to second
order, repeat rate by months since acquisition, before-and-after comparisons around an
event. Order segments are flows measured in a window and can't follow a person forward, so
say plainly that the question needs cohort analysis rather than substituting a
cross-sectional proxy and letting it read as a longitudinal finding.

### Terms that collide with customer segments

Several names appear in both skills meaning different things. When a user says one of
these without context, say which reading you used, or ask if the answer would differ
materially:

| Term | Order reading (here) | Customer reading (`segment-profiles`) |
|---|---|---|
| One-time | An order placed outside a subscription | A customer with exactly one lifetime order |
| Discount | An order carrying a discount | A customer who habitually buys on promotion |
| Subscription | An order billed under a subscription | A customer enrolled in a subscription |
| First order | The order itself | New Customers — the people behind it |
| Repeat | An order from someone who has ordered before | Repeat Customers — order count ≥ 2 |

"Repeat rate" is the one most likely to pass unnoticed, because the two readings are close
on brands with long repurchase cycles and far apart on consumables and subscription brands,
where order-level repeat rate can run near 70% against a customer-level rate near 25%. Say
which denominator you used.

If the user asks for a single number with no breakdown ("how many orders last month?"),
just answer it. If they reference a saved Decile audience, look it up by name and include
the audience id in the query text.

## Workflow

1. **Establish brand context.** Call `get_default_brand` before querying anything. If the
   user names a brand that isn't the default, use `list_brands` with `name_search`. Pass
   `brand_ref` to every `query_data` call exactly as returned — never construct one.

2. **Check the data this analysis depends on.** One cheap query up front covering the
   fields *this particular* question needs — not a fixed checklist. Depending on the
   question that might be the distinct values of `order_source`, whether refund data is
   populated, whether the subscription flags are ever true, whether the segment's defining
   string appears in the catalog, or the labeled breakdown of the dimension you're about to
   split on. Fold the exclusion audit from Which orders count into the same call.

   A dimension with one distinct value, a measure that is uniformly zero, or a string filter
   that matches nothing is **unavailable, not zero**. Report it as not measurable — a 0%
   return rate on a brand with no return data reads as a real finding and is the easiest way
   to mislead someone with this skill. Fields vary by brand, so check rather than assuming
   either way.

3. **Pin down the segment definition.** Use the table below for named segments. Translate
   plain-English descriptions into concrete order-level criteria and state the definition
   in one line. Where a threshold is unset (the inactivity gap for Winback), pick a
   default, proceed, and say which one you used.

4. **Set the time window.** Order segments are flows — an order belongs to the day it was
   placed — so nearly every order question is bounded by a date range. Pass relative
   references ("last 90 days") through verbatim rather than converting to dates. Absent
   user direction, default to trailing 12 months; it's the safer choice because segment
   sizes shrink fast once you filter, and a comparison needs enough orders on the *smaller*
   side to mean anything. Drop to 90 days only when the small side still clears a few
   hundred orders and recency matters to the question. Confirm the brand actually has that
   much history before normalizing anything per day. Exclude the current partial month from
   any trend — a month that's four days old reads as collapse.

5. **Choose the comparison baseline.** Most of these segments come in natural pairs —
   discount/full price, retail/online, weekend/weekday, single/multi-product,
   first/repeat, subscription/one-time. When the user names one side, the other side is a
   sharper baseline than "all orders" because it holds the rest of the business constant.
   When the segment has no counterpart (winback, fully returned, gift-card-only), compare
   against its complement — all valid orders except the segment — while still reporting the
   segment's share against the full total. A baseline that contains the segment is pulled
   toward it by roughly the segment's share, shrinking the gap you're measuring. For a
   dimension with more than two values, report the full mix rather than forcing a binary.

6. **Query, then check the result before interpreting it.** Read the generated SQL, confirm
   the counts reconcile, and confirm the bucket labels are what you asked for. See Getting
   numbers you can trust.

7. **Build the output** following the structure in Output structure.

## Getting numbers you can trust

### Read the SQL it generated

Every `query_data` response includes `generated_sql`. Read it before trusting the result.
It is the only direct evidence that your question was understood the way you meant, it
costs nothing, and it shows you both how the filter was expressed and which columns your
business concepts resolved to. The same question asked twice can generate different SQL, so
read it each time rather than assuming a pattern holds.

**Never split on a numeric date part.** Ask for names wherever one exists — `DAYNAME`
returning `'Sat'`/`'Sun'`, month names over month ordinals — and check the SQL didn't
substitute an ordinal anyway. Day, quarter, and week numbering all vary by convention, and
an off-by-one silently returns the wrong days under a correct-looking label.

A related trap: a customer's order sequence has to be computed over their full history. Rank
inside the filtered window and almost every order looks like someone's first.

### Reconcile counts, and check the labels separately

Ask for the order count alongside every metric, then check that segment plus baseline sums
to the total and that parallel queries agree. This catches dropped rows, join fan-out, and
filter leakage; a disagreement means one query is wrong in a way the numbers alone can't
diagnose.

It does not catch a mislabeled bucket. A binary split built on the wrong filter still sums
to the total, because the two sides are complements by construction. To confirm a bucket
holds what you think, look at its labeled members rather than its total — the day-name
breakdown, the distinct values of the dimension, the product titles that matched.

### Say which revenue you mean

"Revenue" is ambiguous and the difference is material. Gross sales after discounts commonly
resolves to a subtotal that excludes shipping and tax — not what a finance team means by
revenue. Default to gross sales after discounts for consistency with the valid-order rule,
name the measure once in the output, and switch to net sales only when returns are
populated and relevant.

Similarly, "return rate" has two readings that can differ several-fold: share of orders
containing a return, versus returned dollars over gross. Say which one you're reporting.

### Don't report differences you can't support

Below ~30 orders, don't report a rate at all; 30–100, call it directional; above ~100 it can
carry a headline. A 0% discount rate on 51 orders is not a finding.

Size isn't the whole test — what matters is the gap against the spread, which is wide in any
catalog spanning entry-level to premium. Check the gap against the standard error (roughly
the standard deviation over the square root of the order count); inside about two standard
errors, say it's within noise rather than reporting it flat. Corroborate anything you do call
real with a second independent cut — consistency across months, or a percentile breakdown
telling the same story.

Report the median alongside AOV whenever order values span a wide range. A mean gap that
disappears at the median means the segment isn't buying differently, it's buying different
things — often the most informative number in the analysis.

### Which orders count

Cancelled, zero-dollar, marketing/promotional, and draft orders carry revenue that was
never really collected. The First Order and Repeat Order definitions encode the house rule:
**not cancelled, gross sales after discounts > 0**. Apply it to both sides of any revenue
comparison and say that you did. The exception is when those orders *are* the question —
"how many freebies did we send out?" wants exactly what the rule removes.

Verify the exclusion did what you think: query the unfiltered counts once so you can state
how many orders it removed. The share it removes varies enormously by brand, so this is not
something you can assume.

Returns are different. A fully returned order was a real transaction that reversed, so it
belongs in order counts and gross sales but has to be handled explicitly in net revenue.

Finally, watch the null side of a binary. `discount > 0` versus everything else silently
sweeps nulls and negative discounts into "full price." Confirm there are none before
presenting a pair as a clean split.

## Analysis coverage

Order segments are judged on how much they move the business, so lead with size and
economics rather than description.

- **Volume and share** — order count, share of all valid orders, share of revenue. Share of
  revenue diverging from share of orders is itself the finding: 8% of orders and 22% of
  revenue is a different story from 8% of both.
- **Economics** — AOV and median order value, units or line items per order, discount rate
  and depth, gross vs. net sales, return rate where populated.
- **Composition** — which products and categories these orders contain, and whether they
  skew single- or multi-product.
- **Timing and channel** — order source, day or season concentration, trend across the
  window.
- **The customers behind the orders** — distinct customer count, orders per customer within
  the segment, new vs. repeat mix. This is the bridge to `segment-profiles`, which picks up
  the demographic side.

Skip dimensions that are constant by construction — channel mix for a segment defined as
Retail, or day-of-week for a weekend/weekday comparison — and skip dimensions the data
can't support, saying which and why.

### Describing value is not identifying a lever

Order segments tell you which transactions are valuable, not which levers would produce more
of them. The segments differ in what customers chose, so a gap between them is a selection
effect and usually not a treatment effect — full-price orders carry a higher AOV largely
because people buying the premium item don't need a code, which is not evidence that
withdrawing discounts would raise AOV. When the question is prescriptive ("which order types
should we push harder"), report the economics and name the confound rather than converting
the gap into a recommendation.

### Reading shares honestly

Some of these pairs are exhaustive partitions of valid orders: discount/full price,
weekend/weekday, single/multi-product. For those, share of all orders is just the complement
of the other side, and the interesting comparison is against the *structural* expectation
rather than against zero — weekend orders at 27% of volume sounds like a finding until you
notice two of seven days is 28.6%. The same holds past two buckets: across three roughly
equal categories the neutral result is 33%, so 41% is a mild tilt and not a landslide. Say
what the buckets would look like if nothing were distinctive before calling a number high,
and where buckets cover unequal spans, normalize per day before comparing.

Overlap comes in two forms, and neither sums to 100%. Buckets of a single dimension can
overlap — an order holding two product categories lands in both, so the order-share column
runs past 100% while revenue still sums to exactly 100%; side by side they read as if one is
broken, so label the overlapping column rather than letting it pass as a partition. And
different segments overlap freely: one order can be a discounted, online, weekend,
multi-product first order at once, so never present those as summing to 100% or add their
revenue shares together.

## Output structure

1. **Profile summary** — one or two lines on what this kind of order is and why it matters.
   Lead with the finding, not the metric.
2. **Segment analysis** — volume, economics, composition, and timing, with order counts
   visible so magnitude stays honest.
3. **Comparison vs. the baseline** — what actually makes these orders distinct:
   - State which baseline you used and which exclusions were applied.
   - Highlight divergence rather than restating the segment's own numbers.
   - Chart the differentiator that answers the user's question — not necessarily the
     largest gap — as grouped or paired bars. Never stacked; stacking makes the two groups
     impossible to compare. Follow the `dataviz` skill for how to build it. Skip the chart
     when the deliverable is plain text rather than something the user will look at.

Open with the definition you used and the window, in one line, so a wrong assumption is
correctable before the reader gets to the numbers.

For a straightforward mix breakdown ("orders by channel last quarter"), a labelled table or
single chart — chart rules as above — with a short read is the whole deliverable. The full
three-part structure is for segment analysis, not for every order question.

When the data won't support the analysis, don't run the structure anyway. Say what's
unavailable and why, report only what clears the thresholds, and cut the chart rather than
rendering one over numbers that can't bear weight.

## Order segment definitions

Default filter logic for named segments. Thresholds left as variables aren't fixed — use
judgment and call out the value you chose.

| Order Segment Name | Description | Defining Properties / Values |
|---|---|---|
| Discount | Order where a discount was used. | Discount amount > 0. |
| Full Price | Orders where the customer paid full price. | Discount amount = 0. |
| First Subscription Order | The first order placed initiating a subscription agreement. | `is_first_subscription_order` = True. |
| Recurring Subscription Order | An order fulfilled under an active subscription agreement. | `is_subscription_order` = True, `is_first_subscription_order` = False. |
| One Time | One-off purchase made outside of a subscription agreement. | `is_subscription_order` = False. |
| First Order | Initial orders placed by customers. | Not cancelled; gross sales after discounts > 0; first order date. |
| Repeat Order | Orders placed by customers who have ordered before. | Not cancelled; gross sales after discounts > 0; not the customer's first order. |
| Winback | First order after long inactivity. | Days since prior order > threshold. |
| Fully Returned | Orders where all items were returned. | `returned_amount` > 0 and `net_sales` = 0. |
| Partially Returned | Orders where some but not all items were returned. | `returned_amount` > 0 and `net_sales` > 0. |
| Zero Dollar | Orders where all line items are fully discounted, for replacements, marketing, exchanges, and gifting. | `gross_sales_discount_adjusted` = 0. |
| Cancelled | Orders stopped while in process. | `cancelled_date` is not null. |
| Marketing/Promotional | Orders given for free for marketing or promotional purposes. | Order tags contain marketing, influencer, or promotion, and `gross_sales_discount_adjusted` = 0. |
| Retail | Orders placed in a retail location. | `order_source` = 'POS'. |
| Online | Orders placed online. | `order_source` = 'Web'. |
| Draft | Orders that were created manually. | `order_source` = 'shopify_draft_order'. |
| Gift Card Only Order | A gift card was the only item purchased. | Product title includes gift card; only 1 line item. |
| Includes Bundles | Orders that include product bundles. | 'Bundle' in product title. |
| Multi-product orders | Orders that include multiple products. | Multiple line items. |
| Single-product orders | Orders that only include a single product. | Single line item. |
| Weekend Orders | Orders placed on a weekend. | Placed Saturday or Sunday — match on day names, never day numbers, and note that day boundaries follow the warehouse timezone rather than the customer's. |
| Weekday Orders | Orders placed during the week. | Placed Monday–Friday, same caveats. |

Three groups here depend on values that vary by brand and need the availability check in
step 2 before use: Retail, Online, and Draft depend on `order_source` carrying readable
labels; Fully and Partially Returned depend on return data being populated; Includes Bundles
and Gift Card Only depend on the catalog actually using those words in product titles.

## Notes

- Not an exhaustive list. If a user asks about a transaction type that doesn't map cleanly
  to the table, define it in plain language, state your assumptions, and proceed.
- The Discount definition catches order-level discount codes. It misses sale-priced SKUs,
  permanent markdowns, and compare-at pricing, so a brand running a standing promotion can
  show a low discount rate. Flag this whenever the question is about promotional dependence
  rather than about discount codes specifically.
- Don't call the output a "persona." Personas is a named Decile feature; use profile,
  archetype, or order type instead.
- Decile data runs on a 24-hour delay, so today's orders aren't included.
