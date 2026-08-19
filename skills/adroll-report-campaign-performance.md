---
name: adroll-report-campaign-performance
description: Pull AdRoll campaign, adgroup and ad performance in a single call with the GraphQL Reporting API.
api: NextRoll GraphQL Reporting API
base_url: https://services.adroll.com
operations:
  - POST /reporting/api/v1/query
schema: graphql/adroll-reporting.graphql
generated: '2026-08-13'
method: generated
source: https://apidocs.nextroll.com/graphql-reporting-api/overview.html
---

# Report on AdRoll performance

Use the **GraphQL Reporting API** — not the REST `/report` endpoints. NextRoll
states the GraphQL API *replaces* `/report` on the CRUD API and `/report` +
`/metrics` on the Prospecting API.

**Endpoint:** `POST https://services.adroll.com/reporting/api/v1/query?apikey=MYAPIKEY`
**Console:** <https://app.adroll.com/reporting/graphiql> (read-only, safe to explore)

## Query shape

Roots on `Query`: `organization`, `advertisable`, `campaign`, `adgroup`, `ad`,
`automation`, `email`, `segment`, `group`, `log`.

Entry points that actually resolve entities:

- `advertisable.forUser` → `[Advertisable]!` — everything you can see
- `advertisable.byEID(advertisable: String!)` → `Advertisable`
- `campaign.byAdvertisable(advertisable: String!, statuses: [String!], channels: [String!], …)` → `[Campaign]!`
- `campaign.byEID(campaign: String!)` → `Campaign`
- `organization.current` → `Organization`

## Worked example

```bash
curl -H 'Authorization: Token MYTOKEN' \
     -H 'Content-Type: application/json' \
     -d '{"query":"query Perf { advertisable { forUser { eid name campaigns { eid name metrics(start: \"2026-07-01\", end: \"2026-08-01\") { summary { impressions clicks cost conversions ctr cpc cpa roas } } } } } }"}' \
     'https://services.adroll.com/reporting/api/v1/query?apikey=MYAPIKEY'
```

`metrics(start:, end:, pastDays:, currency:, duration:)` returns a
`MetricResult!` with two selections:

- `summary: Metric!` — one aggregate row
- `byDate: [Metric!]!` — a daily time series

`Metric` fields you will actually want (all `Decimal`): `impressions`, `clicks`,
`cost`, `conversions`, `clickThroughs`, `viewThroughs`, `revenue`, `ctr`, `cpc`,
`cpm`, `cpa`, `clickCPA`, `viewCPA`, `averageOrderValue`, `roas`, `roi`,
`audienceSizeTotal`, `bounceRate`. The full 107-type schema is in
[`graphql/adroll-reporting.graphql`](../graphql/adroll-reporting.graphql).

## Read the errors — they are NOT standard GraphQL

```json
{
  "has_errors": true,
  "errors": [ { "id": "E001", "msg": "HTTP request failed" } ],
  "request": "req46209",
  "version": "2018.09.11-1",
  "data": { "advertisable": { "has_errors": true, "errors": ["E001"], "byEID": { "campaigns": [] } } }
}
```

1. Check the **root `has_errors`** before you touch `data`.
2. Object-level `errors[]` hold **IDs**; the human message lives in the root
   `errors[]` keyed by that ID.
3. `has_errors: true` propagates up to the root from any failing nested object.
4. Partial data is returned alongside errors — either handle it explicitly or
   discard the whole response. Do not render a mix.
5. Keep the `request` id (`req46209`); support asks for it.

## Timing

All dates and times are **UTC**. Same-day numbers appear within roughly twelve
hours of the day ending and are **not final until 48 hours** after the start of
the current UTC day. Do not treat today's figures as settled.

## Efficiency

There is no limit on GraphQL query size, and NextRoll fans out to its backing
services in parallel. One large query across many advertisables is both faster
and cheaper against the daily per-service quota than many small ones.
