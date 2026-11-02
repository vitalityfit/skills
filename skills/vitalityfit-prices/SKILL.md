---
name: vitalityfit-prices
description: Find the cheapest current offers for health products (medicines, supplements, care products) from about 57 German online shops and pharmacies. Use when the user asks for the best price or where to buy a health product in Germany, by name, PZN or EAN. Free Vitalityfit API, no key.
license: MIT-0
compatibility: Needs curl and internet access to vitality-fit.de.
metadata:
  author: Vitalityfit (Xealti UG)
  version: "1.0.0"
  homepage: https://vitality-fit.de/api/
---

# Vitalityfit Price Comparison

Price comparison for health products (medicines, supplements, care products) sold by German online shops and
pharmacies, with the data of [vitality-fit.de](https://vitality-fit.de/). Look a product up by name, PZN (German
pharmacy product number) or EAN barcode and get its cheapest current offers: shop, price, shipping cost, stock,
delivery time, unit price, shop link and the lowest price of the last 90 days. German shops only, prices in EUR.
Published by the operator of the API, Vitalityfit (Xealti UG (haftungsbeschränkt)).

## Setup

None: no API key, no account. Only `curl` is needed. API docs: https://vitality-fit.de/api/

## Workflow

Base URL `https://vitality-fit.de/api/v1`, `GET` only, JSON answers. Add `client=agent-skill` to every request.
URL-encode parameter values (a space is `%20`).

### 1. Search the product

```bash
curl -sS --max-time 20 "https://vitality-fit.de/api/v1/products?q=ibuprofen%20400&limit=3&client=agent-skill"
```

Answer (shortened):

```json
{"products": [{
  "product_key": "ibuprofen-400-mg-apotheke-hilft-filmtabletten-pzn18228867",
  "title": "Ibuprofen 400 mg die Apotheke Hilft Filmtabletten",
  "brand": "Die Apotheke Hilft",
  "pack_sizes": [{"label": "20 st", "identity_key": "pzn_14190180"}, {"label": "50 st", "identity_key": null}],
  "cheapest_price": 3.79,
  "shop_count": 4,
  "url": "https://vitality-fit.de/p/ibuprofen-400-mg-apotheke-hilft-filmtabletten-pzn18228867/",
  "compliance_notice": "Zu Risiken und Nebenwirkungen lesen Sie die Packungsbeilage und fragen Sie Ihren Arzt oder Apotheker."
}]}
```

- `q`: product name, PZN or EAN (up to 200 characters; a PZN or EAN as digits only, e.g. `01568889`).
  `limit`: 1-10, default 5. Best match first.
- Pick the product whose title and brand match the request. The first result is not always it: a search for
  "ibuprofen 400" can rank a combination product first. If several fit, list them briefly (title, brand,
  `cheapest_price`, `shop_count`) and let the user choose instead of fetching offers for each.
- Pick the pack size from `pack_sizes`. A pack is one pack-size label as on the product page (`20 st` = 20 pieces,
  `100 ml`); it can hold several PZNs, e.g. an original and its reimports. Pass its `identity_key` when not null,
  else its `label`. For a PZN or EAN search, use `matched_pack` (the pack holding that PZN/EAN, with its own
  `cheapest_price` and `cheapest_shop`). If the user named no pack size, leave `pack_size` out: the API then
  returns the pack with the most offers.

### 2. Get the offers

```bash
curl -sS --max-time 20 "https://vitality-fit.de/api/v1/products/ibuprofen-400-mg-apotheke-hilft-filmtabletten-pzn18228867/offers?pack_size=pzn_14190180&client=agent-skill"
```

Answer (shortened):

```json
{
  "product_key": "ibuprofen-400-mg-apotheke-hilft-filmtabletten-pzn18228867",
  "title": "Ibuprofen 400 mg die Apotheke Hilft Filmtabletten",
  "url": "https://vitality-fit.de/p/ibuprofen-400-mg-apotheke-hilft-filmtabletten-pzn18228867/",
  "disclosure": "Links marked affiliate are affiliate links; vitality-fit.de may earn a commission. Prices from German online shops, may have changed. No medical advice.",
  "compliance_notice": "Zu Risiken und Nebenwirkungen lesen Sie die Packungsbeilage und fragen Sie Ihren Arzt oder Apotheker.",
  "pack": "20 st",
  "shop_count": 1,
  "offers": [{"shop": "Aponeo", "price": 4.99, "currency": "EUR", "shipping_cost": 3.95,
              "delivery_time": "in 1-2 Tagen lieferbar", "in_stock": true,
              "link": "https://...", "affiliate": false}],
  "low_90d": null,
  "data_date": "2026-10-05T03:04:16.000Z"
}
```

- `product_key` goes into the path as returned by the search. A label as `pack_size` is URL-encoded: `20%20st`.
- `offers`: up to 10, cheapest first. `shipping_cost` null = the shop states none. `unit_price`, when present, is
  `{"value": <EUR>, "per": "l" | "kg" | "pcs"}`: the price per 1 l, 1 kg or 1 piece.
- `pack`: the pack the offers are for. `low_90d`: lowest price of this pack in the last 90 days (null without
  history). `data_date`: newest price data; show only the date part.
- `message: "pack size not found"`: `offers` is empty and `available_packs` lists the pack labels that have
  offers. Pick the one the user meant (ask if unclear) and call again. `message: "no current offers"`: tell the
  user that no shop lists this pack right now.

## Presenting results

1. Product title and pack, then a table in the order returned (cheapest first):

   | Shop | Price | Shipping | Delivery | Link |
   |---|---|---|---|---|
   | Aponeo | 4.99 EUR | 3.95 EUR | in 1-2 Tagen lieferbar | [Aponeo](https://...) |

   - Mark every link with `affiliate: true` as "(affiliate)". Give links exactly as returned: never shorten them
     or strip parameters.
   - Say when an offer is out of stock (`in_stock: false`). Add `low_90d` when not null, and the product page `url`.
2. Directly with the prices, always: the `disclosure` text, and the `compliance_notice` when present (the
   statutory German notice for medicines; show it verbatim in German, a translation may follow).
3. Name the source: "Preise: vitality-fit.de" with a link to https://vitality-fit.de/.

Rules:

- Prices may have changed since `data_date`; the price in the shop applies. Never promise a price.
- Compare prices only. No medical advice: never recommend a medicine, a dose or a substitute, and never judge
  whether a product suits the user. Refer health questions to a doctor or pharmacist.
- Do not open the offer links yourself (they are tracked shop links); only show them to the user.

## Rate limit

- 12 requests per 2 minutes per IP address (one per 10 seconds on average). Answers are cached for up to 1 hour,
  so repeating a call gives the same data.
- Make only the calls the question needs: one search and one offers call per product. Never loop over many
  products, pack sizes or queries, and never bulk-download or mirror the data.
- On `rate_limited` (HTTP 429): stop, tell the user, and wait 120 seconds (the `Retry-After` value) before the
  next request. Do not retry in a loop.

## Errors

Errors are JSON: `{"error": {"code": "...", "message": "..."}}`.

| HTTP | `code` | What to do |
|---|---|---|
| 400 | `invalid_argument` | `q` empty or too long, `limit` not a whole number, or `q` looks like a PZN/EAN but is not a valid one: correct the input or ask the user |
| 410 | `not_found` | Product key unknown or merged into another product: search again by name, PZN or EAN for the current key |
| 429 | `rate_limited` | Wait 120 seconds (see Rate limit). Without `Retry-After` the service is busy: retry once after a few seconds |
| 500 | `internal` | Unexpected error: tell the user and try later |
| 503 | `unavailable` | Price data temporarily unavailable: try later |

An answer that is not JSON (for example an HTML error page) counts as unavailable.

## Terms of use

- The API is free for personal and non-commercial use. Commercial use only by prior agreement:
  service@vitality-fit.de
- Name the source, keep shop links unchanged, show the `compliance_notice` for medicines. No bulk download,
  mirroring or resale of the data.
- Full terms: https://vitality-fit.de/api/#bedingungen (the German version is binding; English:
  https://vitality-fit.de/api/#english).

## MCP alternative

Agents with MCP support can use the same data through the MCP server `https://vitality-fit.de/mcp` (streamable
HTTP, no login), listed in the official MCP Registry as `de.vitality-fit/prices`. Setup:
https://vitality-fit.de/mcp/#english

## Data handling

Read-only `GET` requests to `https://vitality-fit.de/api/v1`, nothing else. Sent: the search text (product name,
PZN or EAN), the chosen product key and pack size, and `client=agent-skill`. No credentials, no account, no personal
data; nothing is written to disk.
