---
name: benchmark-minerals-track-composite-index
description: Track Benchmark composite price indices across a market or across the battery raw-material
  basket, for contract indexation and portfolio benchmarking.
api: Benchmark Mineral Intelligence Prices API v2
base_url: https://api.benchmarkminerals.com/v2
spec: https://www.benchmarkminerals.com/prices-api.yml
operations:
- lithium-index
- cobalt-index
- nickel-index
- natural-graphite-index
- synthetic-graphite-index
- anode-index
- cathodes-index
- lithium-ion-battery-index
- black-mass-index
- rare-earths-index
- lithium-ion-battery-raw-material-index
generated: '2026-09-04'
method: generated
source: openapi/benchmark-minerals-prices-api.yml
---

# benchmark-minerals-track-composite-index

Track Benchmark composite price indices across a market or across the battery raw-material basket, for contract indexation and portfolio benchmarking.

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

An index operation returns a composite series across several grades rather than one grade at a time. This is the surface to use for contract indexation clauses and for tracking a basket.

## Steps

1. **Choose the index operation.** Eleven exist: `lithium-index`, `cobalt-index`, `nickel-index`, `natural-graphite-index`, `synthetic-graphite-index`, `anode-index`, `cathodes-index`, `lithium-ion-battery-index`, `black-mass-index`, `rare-earths-index`, and the cross-market `lithium-ion-battery-raw-material-index` (`/prices/lithium-ion-battery-raw-material/index`).

2. **Note which markets have no index.** Manganese, electrolyte and spotlight expose list and/or summary operations only. If a user asks for a manganese index, say it is not published rather than constructing one from grades.

3. **POST the index filter:**

   ```json
   { "from": "2023-01-01", "to": "2023-12-31", "indexes": ["Lithium Carbonate", "Lithium Hydroxide", "Lithium"] }
   ```

   `indexes[]` names which composites to return. Omit it for everything your subscription covers.

4. **Read the result.** Index responses carry a `DataSummaryIndex`: `today`, `last`, `latestChange`, `yearOnYear`, `yearToDate` — a different and smaller set than the grade-level `DataSummary`. Do not expect `dailyChange` or `monthlyChange` here.

## Cautions

- These indices are used as contract reference points. If a value is null for a date, report the gap; never carry the last value forward and present it as the index for that day.
- Index membership is Benchmark's methodology, published under the price assessment control documents at https://www.benchmarkminerals.com/price-assessments. Do not reconstruct or re-weight an index yourself and call it a Benchmark index.
