# Vitalityfit skills

Agent skill and Claude Code plugin for [Vitalityfit](https://vitality-fit.de/), a price comparison for health
products (medicines, supplements, care products) sold by German online shops and pharmacies. Look a product up by
name, PZN (German pharmacy product number) or EAN barcode and get its cheapest current offers: shop, price,
shipping, stock, delivery time, unit price, shop link and the lowest price of the last 90 days. Free, read-only,
no API key.

| | What it does | Folder |
|---|---|---|
| Agent skill `vitalityfit-prices` | Teaches an agent to use the public price API with `curl` | [`skills/vitalityfit-prices`](skills/vitalityfit-prices/SKILL.md) |
| Claude Code plugin `vitalityfit-prices` | Adds the Vitalityfit MCP server to Claude Code | [`plugins/vitalityfit-prices`](plugins/vitalityfit-prices) |

## Install

### Agent skill (Claude Code, Codex, Cursor and other agents)

```sh
npx skills add vitalityfit/skills
```

Or copy `skills/vitalityfit-prices/` into your agent's skills folder (Claude Code: `~/.claude/skills/`).

### Claude Code plugin

```
/plugin marketplace add vitalityfit/skills
/plugin install vitalityfit-prices@vitalityfit
```

Without the plugin: `claude mcp add --transport http vitalityfit https://vitality-fit.de/mcp`

### Other MCP clients

Server URL `https://vitality-fit.de/mcp` (streamable HTTP, no login), listed in the official MCP Registry as
`de.vitality-fit/prices`. Setup for ChatGPT, Claude and other clients: https://vitality-fit.de/mcp/#english

## Try it

> Where is Ibuprofen 400 mg (20 tablets) cheapest right now?

> Compare prices for PZN 03464237.

The agent searches the product, picks the pack size and shows the offers cheapest first, with the shop links,
the affiliate disclosure and, for medicines, the statutory German notice ("Zu Risiken und Nebenwirkungen ...").

## Data and terms

- German online shops only, prices in EUR. Prices may have changed since the data date; the price in the shop
  applies.
- Price comparison only, no medical advice.
- Links marked affiliate are affiliate links; vitality-fit.de may earn a commission.
- The API is free for personal and non-commercial use; commercial use by prior agreement
  (service@vitality-fit.de). Rate limit: 12 requests per 2 minutes per IP. Terms:
  https://vitality-fit.de/api/#english

## More

- API docs: https://vitality-fit.de/api/ (OpenAPI: https://vitality-fit.de/api/v1/openapi.json)
- Client libraries: [`vitalityfit` on npm](https://www.npmjs.com/package/vitalityfit) and
  [`vitalityfit` on PyPI](https://pypi.org/project/vitalityfit/)
- Contact: service@vitality-fit.de

## License

The files in this repository are licensed under [MIT-0](LICENSE). The Vitalityfit name and logo are not covered by
the license. The price data is governed by the [API terms](https://vitality-fit.de/api/#english).
