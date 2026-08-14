---
name: customer-segments
description: "Profile and analyze customer segments using Decile's customer intelligence MCP tools. Use this whenever the user asks a broad question about a customer segment or cohort — e.g. \"who are our VIP customers\", \"profile our repeat customers\", \"what does our churn-risk segment look like\", \"break down high AOV customers vs everyone else\", \"tell me about our discount-driven buyers\" — even if they don't name a segment from the standard list below, or ask about a custom-defined group. Also use this to figure out how to translate a plain-English segment description (like \"customers who buy every month\" or \"big spenders who never use coupons\") into concrete Decile query filters, and to structure the resulting analysis into a segment profile summary, metric breakdown, and comparison against the full customer base."
---

# Customer Segments

## When to use this skill

Reach for this skill any time someone asks a broad, open-ended question about a group of customers rather than a single narrow metric. Signals this is the right skill:

- A named segment from the definitions table below ("VIP customers," "lapsed customers," "one-time buyers," etc.)
- A plain-English description of a cohort that needs to be translated into filter logic ("people who bought more than 3 times but never used a discount")
- A request to "profile," "describe," "characterize," or "give me a breakdown of" any group of customers or contacts
- A request to compare a segment against the overall customer base

If the user instead asks for a single specific number (e.g. "what's our average order value this month?"), just answer that directly — this skill is for the fuller, persona-style analysis, not one-off metric lookups.

If the user asks about a saved audience (a predefined list or segment saved from their Decile account) look up that audience by name and include the audience id in query prompts. If it is unclear if the user is referencing one of the common segments below or a saved audience, use the ask_user tool to clarify before proceeding.

## Workflow

1. **Establish brand context.** Call `get_default_brand` (or `list_brands` / `switch_brand` if the user references a brand other than the default, or works across  multiple brands) before querying anything. Every `query_data` call needs to run against the right brand's data.

2. **Pin down the segment definition.** If the user names one of the segments in the table below, use its defining properties as the filter logic. If they describe a custom segment in plain English, translate it into concrete filter criteria yourself (order count, recency, revenue percentile, discount usage rate, etc.) and briefly state the definition you used so the user can correct it if you assumed wrong. Where a segment definition depends on a threshold Decile hasn't told you (e.g. "recent" for New Customers, or the percentile cutoff for VIPs), pick a sensible default (30/60/90 days for recency, top 5-10% for VIP tiers, etc.) and say so in the output rather than blocking on it.

3. **Pull the data with `query_data`.** Query both the segment itself and the full customer base, since the output requires a side-by-side comparison. Gather the metrics listed under Analysis below for each.

4. **Build the output** following the exact structure in the Output section.

## Analysis coverage

For any segment profile, pull and cover:

- **Demographics** — age, gender, income, education, population density
- **Behavioral metrics** — average lifetime revenue (LTR), average order value (AOV),  average order count, average customer tenure (days between acquisition and today)
- **Product preferences** — categories or products the segment favors

## Output structure

Always structure the final response as:

1. **Profile summary** — a high-level, holistic, persona-style description of the segment in 1-2 lines. Write it like a quick character sketch, not a data dump.
2. **Segment analysis** — the demographic, behavioral, and product-preference results for the segment. 
3. **Comparison vs. all customers** — this is the section that actually answers "what makes this segment distinct":
   - Highlight where the segment diverges from the overall customer base, not just restate its own numbers.
   - Make sure the comparison directly answers: what are the defining characteristics of this segment vs. the entire customer base?
   - Pick the single most interesting differentiator and visualize the segment vs. all customers side-by-side (grouped/paired bars, not a stacked chart — the two groups need to stay visually separable).

## Segment definitions

Use these as the default filter logic when a user names one of these segments. Thresholds marked with a variable (X) aren't fixed — use good judgment or ask the user, defaulting to a reasonable value. If you choose the threshold, call it out in the response.

| Segment Name | Description | Defining Properties / Values |
|---|---|---|
| Purchasers | Customers who have purchased at least once. | Order count ≥ 1. |
| Leads | Contacts who have not made a purchase. | Order count = 0. |
| New Customers | Customers who have made their first purchase recently. | Order count = 1; first purchase date within last X days. |
| Repeat Customers | Customers who have purchased more than once. | Order count ≥ 2. |
| Active Customers | Customers who purchased within a recent timeframe. | Last order date within last X days. |
| Lapsed Customers | Customers who haven't purchased in a while. | Last order date > X days ago. |
| High Value Customers | Customers who generate high revenue. | Top 20% of customers by lifetime revenue. |
| One-Time Buyers | Customers who purchased only once. | Order count = 1; first purchase date not recent. |
| VIP / Top Customers | The most valuable or loyal customers. | Top 5–10% by lifetime revenue, order count, or RFM score. |
| Churn-Risk Customers | Previously active customers showing signs of inactivity. | Last purchase approaching lapsed threshold. |
| Discount-Driven Customers | Customers who usually buy with promotions. | Multiple orders where % of orders with discount ≥ threshold. |
| Full-Price Customers | Customers who rarely or never use discounts. | Multiple orders where % of orders with discount ≤ threshold. |
| High AOV Customers | Customers with high average order value. | Average order value above threshold or percentile. |
| Frequent Buyers | Customers who purchase often. | Order count ≥ threshold. |
| Recently Acquired Customers | Customers acquired within a recent period. | First purchase date within last X days/weeks. |
| Channel-Specific Customers | Customers primarily acquired through a specific channel. | First-touch or majority of orders from a channel. |
| Category Loyalists | Customers who repeatedly purchase from a specific category. | Multiple orders with majority of orders or spend in one category. |
| Subscription Customers | Customers enrolled in a subscription program. | Active subscription status or subscription order history. |
| Seasonal or Occasional Buyers | Customers who buy only during certain events or seasons. | Purchases concentrated in specific months or events. |
| Refund/Return-Prone Customers | Customers with high return or refund rates. | Multiple orders with refund rate or returned-item % above threshold. |
| At-Risk High-Value Customers | Valuable customers who haven't purchased recently. | High lifetime value + last purchase > X days ago. |

## Notes

- This document is not an exhaustive list of every possible segment or analysis — if a user asks about a cohort that doesn't map cleanly to the table, define it in plain language, state your assumptions, and proceed rather than blocking on a perfect match.
- Always run the comparison step (vs. all customers). A segment profile without that comparison doesn't answer the question a persona-style analysis is meant to answer.