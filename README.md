# Decile Plugin for Claude

A Claude plugin that connects the [Decile](https://www.decile.com) customer intelligence MCP with Decile analysis skills, so teams can explore their store's performance and customers in plain language — in chat, Cowork, or anywhere else Claude runs.

## What's included

**Decile MCP** — the Decile connector (`https://mcp.decile.com/mcp`). Decile is an ecommerce customer intelligence platform; the connector puts analyst-level insights and recommendations in the hands of your team. Ask about revenue, orders, units, AOV, and profit; product and category performance; customer acquisition, retention, cohorts, and subscriptions. Deep dive into market basket analysis or sequential purchase flows, describe a group of customers to profile how they differ from your base, and save that group to your Decile account as a segment for reuse and activation. Requires authorizing with your Decile account on first use.

**Skills** — guided workflows Claude uses automatically when a question matches:

| Skill | Use it for |
|---|---|
| [customer-segments](skills/customer-segments/SKILL.md) | In-depth profiles of customer cohorts ("who are our VIP customers?"), with standard segment definitions and comparison against the full customer base. |
| [order-segments](skills/order-segments/SKILL.md) | Order-level analysis where the unit is an order — discount share, subscription vs one-time mix, first vs repeat orders, and which orders to exclude as non-revenue noise. |
| [us-population-comparison](skills/us-population-comparison/SKILL.md) | Benchmarking customer demographics and interests against US population distributions — over/under-indexing for audience insight and ad targeting. |

## Installation

Install the Decile plugin from the Claude plugin directory (in Claude, browse the plugin directory and add **Decile**). On first use, Claude will prompt you to authorize the connection with your Decile account.

Once installed, just ask questions in chat or Cowork — no setup beyond authorization. For example:

- "How did revenue and AOV trend last quarter?"
- "Who are our VIP customers, and how do they differ from everyone else?"
- "What share of our orders used a discount?"
- "How do our customers compare to the average American?"

## Layout

```
.claude-plugin/
  plugin.json        # plugin manifest
  marketplace.json   # marketplace listing metadata
.mcp.json            # Decile MCP server definition
skills/
  customer-segments/SKILL.md
  order-segments/SKILL.md
  us-population-comparison/SKILL.md
```
