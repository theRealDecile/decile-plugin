# Decile Plugin for Claude

A Claude Code plugin that connects the [Decile](https://www.decile.com) customer intelligence MCP server and packages the Decile analysis skills.

## What's included

**MCP server** — `decile`, the Decile customer intelligence MCP (`https://mcp.decile.com/mcp`). Exposes brand data, audiences, personas, `query_data`, and the Ask Luma agent. Requires OAuth authorization on first use.

**Skills**

| Skill | Use it for |
|---|---|
| [customer-segments](skills/customer-segments/SKILL.md) | Persona-style profiles of customer cohorts ("who are our VIP customers?"), with standard segment definitions and a required comparison against the full customer base. |
| [order-segments](skills/order-segments/SKILL.md) | Analysis where the unit is an order, not a person — discount share, subscription vs one-time mix, first vs repeat orders, and which orders to exclude as non-revenue noise. |
| [us-population-comparison](skills/us-population-comparison/SKILL.md) | Benchmarking customer demographics and interests against US population distributions — over/under-indexing for audience insight and ad targeting. |

## Installation

Add this repo as a plugin marketplace, then install the plugin:

```bash
claude plugin marketplace add <repo-url-or-local-path>
```

```bash
claude plugin install decile@decile
```

On first use of a Decile tool, Claude Code will prompt you to authorize the MCP server via OAuth (`/mcp` in an interactive session).

## Layout

```
.claude-plugin/
  plugin.json        # plugin manifest
  marketplace.json   # makes this repo directly installable as a marketplace
.mcp.json            # Decile MCP server definition
skills/
  customer-segments/SKILL.md
  order-segments/SKILL.md
  us-population-comparison/SKILL.md
```
