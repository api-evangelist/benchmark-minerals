---
name: benchmark-minerals-pull-price-series
description: Pull an assessed price time series for any Benchmark market (lithium, cobalt, nickel, graphite,
  cathode, anode, battery cells, black mass, rare earths and more) over a date window, filtered by category,
  purity, shipping route and price type.
api: Benchmark Mineral Intelligence Prices API v2
base_url: https://api.benchmarkminerals.com/v2
spec: https://www.benchmarkminerals.com/prices-api.yml
operations:
- lithium-list-prices
- cobalt-list-prices
- nickel-list-prices
- manganese-list-prices
- natural-graphite-list-prices
- synthetic-graphite-list-prices
- electrolyte-list-prices
- anode-list-prices
- cathodes-list-prices
- lithium-ion-battery-list-prices
- black-mass-list-prices
- rare-earths-list-prices
- spotlight-list-prices
generated: '2026-09-04'
method: generated
source: openapi/benchmark-minerals-prices-api.yml
---

# benchmark-minerals-pull-price-series

Pull an assessed price time series for any Benchmark market (lithium, cobalt, nickel, graphite, cathode, anode, battery cells, black mass, rare earths and more) over a date window, filtered by category, purity, shipping route and price type.

## Ground rules for every call

- Base URL: `https://api.benchmarkminerals.com/v2`
- Auth: send your key as the `x-api-key` request header. Get it from the subscription section of your Benchmark account (https://www.benchmarkminerals.com/api). There is no public sign-up.
- Every operation is a **POST** that carries a JSON filter object. There are no GET operations, no path parameters and no query parameters. Despite the verb, nothing you call here changes state — the whole surface is read-only.
- Dates are strings in `yyyy-MM-dd`. `from` and `to` bound the series window.
- There is **no pagination**. Narrow with the date window and the filter arrays; do not expect a cursor or a next page.
- **Do not retry blindly.** Benchmark applies rate limits but publishes no numbers, no `RateLimit-*` headers and no 429 in the contract. Exceeding limits is documented as risking "temporary suspension of access", so back off conservatively and cache aggressively.
- Errors: a `400` returns a bare JSON **array** of `{type, msg, path, location}` validation objects — not RFC 9457 problem+json. A `401` means the key is missing or unrecognised; a `403` means the key is valid but the subscription does not cover that market. Neither carries a response body schema, so surface the status code to the user rather than trying to parse a reason.
- Entitlement is per market. Do not assume a working key for lithium works for rare earths.

## What this does

Returns the full assessed price history for one market: a list of graded products, each with its category, purity, shipping route, unit of measure, currency and price type, plus a `series[]` of dated `{valueLow, valueMid, valueHigh}` observations.

## Steps

1. **Pick the market operation.** One POST per market:

   | Market | Operation | Path |
   |---|---|---|
   | Lithium | `lithium-list-prices` | `/prices/lithium` |
   | Cobalt | `cobalt-list-prices` | `/prices/cobalt` |
   | Nickel | `nickel-list-prices` | `/prices/nickel` |
   | Manganese | `manganese-list-prices` | `/prices/manganese` |
   | Natural graphite | `natural-graphite-list-prices` | `/prices/natural-graphite` |
   | Synthetic graphite | `synthetic-graphite-list-prices` | `/prices/synthetic-graphite` |
   | Electrolyte | `electrolyte-list-prices` | `/prices/electrolyte` |
   | Anodes | `anode-list-prices` | `/prices/anodes` |
   | Cathodes | `cathodes-list-prices` | `/prices/cathodes` |
   | Lithium-ion battery | `lithium-ion-battery-list-prices` | `/prices/lithium-ion-battery` |
   | Black mass | `black-mass-list-prices` | `/prices/black-mass` |
   | Rare earths | `rare-earths-list-prices` | `/prices/rare-earths` |
   | Spotlight | `spotlight-list-prices` | `/prices/spotlight` |

2. **Build the filter body.** The exact fields vary by market, so read the request DTO for the operation you chose before assuming a field exists. The common shape:

   ```json
   {
     "from": "2023-01-01",
     "to": "2023-12-31",
     "categories": [{ "name": "Lithium Carbonate", "isSustainable": false }],
     "purities": ["Min 99.5%"],
     "shippingRoutes": ["CIF Asia"],
     "priceTypes": ["Price"]
   }
   ```

   Omit a filter array to get everything you are entitled to in that dimension. Send `from`/`to` on every call — without a window you will pull the entire history.

3. **POST it** to `https://api.benchmarkminerals.com/v2{path}` with `x-api-key` and `Content-Type: application/json`.

4. **Read the response.** `data[]` is the grade list. Join on the reference objects rather than the display strings: `category.id`, `purity.id`, `shippingRoute.id` and `priceType.id` are UUIDs and are the stable keys. `name` and `alias` are labels and can change.

5. **Handle the failure modes.** `400` → read the array and fix the named `path` field. `403` → the subscription does not cover this market; stop and tell the user, do not retry. `401` → the key is wrong.

## Cautions

- `valueLow`/`valueHigh` are nullable. A grade assessed as a single point returns `valueMid` only — do not compute a spread from nulls.
- Benchmark prices are IOSCO-assured reference prices used in real supply contracts and in ICE-listed futures settlement. Never present a value you interpolated, smoothed or forward-filled as a Benchmark assessment.
