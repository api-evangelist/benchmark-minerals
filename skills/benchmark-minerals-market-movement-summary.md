---
name: benchmark-minerals-market-movement-summary
description: Get the current level and the daily, bi-weekly, monthly and latest percentage change for
  grades in a Benchmark market, without pulling the full price history.
api: Benchmark Mineral Intelligence Prices API v2
base_url: https://api.benchmarkminerals.com/v2
spec: https://www.benchmarkminerals.com/prices-api.yml
operations:
- lithium-summary
- cobalt-summary
- nickel-summary
- manganese-summary
- natural-graphite-summary
- synthetic-graphite-summary
- electrolyte-summary
- anode-summary
- cathodes-summary
- lithium-ion-battery-summary
- black-mass-summary
- rare-earths-summary
generated: '2026-09-04'
method: generated
source: openapi/benchmark-minerals-prices-api.yml
---

# benchmark-minerals-market-movement-summary

Get the current level and the daily, bi-weekly, monthly and latest percentage change for grades in a Benchmark market, without pulling the full price history.

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

Every market exposes a `/summary` sibling that returns change statistics instead of the raw series. Use it when the question is "what moved and by how much", not "give me the history" — it is a far smaller response than the list operation.

## Steps

1. **Choose the summary operation** for the market: `lithium-summary` (`/prices/lithium/summary`), `cobalt-summary`, `nickel-summary`, `manganese-summary`, `natural-graphite-summary`, `synthetic-graphite-summary`, `electrolyte-summary`, `anode-summary`, `cathodes-summary`, `lithium-ion-battery-summary`, `black-mass-summary`, `rare-earths-summary`.

2. **POST the same filter shape** you would send to the list operation. Keep the window tight — the summary is computed relative to the window you give it.

3. **Read the `DataSummary`** on each grade:

   | Field | Meaning |
   |---|---|
   | `last` | the previous value |
   | `latestChange` | change against the most recent prior assessment |
   | `dailyChange` | change vs the previous day |
   | `biWeeklyChange` | change vs two weeks ago |
   | `monthlyChange` | change vs a month ago |

   Each change object is `{value, unit}` where `unit` is `%`.

4. **Every change field is nullable.** A grade with no prior observation in the window returns null. Render "no prior assessment", never zero — a fabricated 0% move reads as a stable market.

## Comparing markets

There is no cross-market operation. To compare lithium against cobalt, call both summaries and align them yourself on the date window. Do not assume the two markets publish on the same schedule; assessment frequency differs by grade.
