---
generated: '2026-08-13'
method: searched
source: https://apidocs.nextroll.com/graphql-reporting-api/overview.html
---

# NextRoll GraphQL Reporting API (AdRoll)

AdRoll's parent, NextRoll, publishes a real GraphQL API — the **GraphQL Reporting
API** — alongside its REST services. It is the sanctioned way to retrieve
reporting data and it explicitly **replaces** the `/report` endpoints of the CRUD
API and the `/report` and `/metrics` endpoints of the Prospecting API.

| | |
|---|---|
| **Endpoint** | `POST https://services.adroll.com/reporting/api/v1/query` |
| **Console** | https://app.adroll.com/reporting/graphiql (GraphiQL) |
| **Overview** | https://apidocs.nextroll.com/graphql-reporting-api/overview.html |
| **Schema reference** | https://apidocs.nextroll.com/graphql-reporting-api/schema.html |
| **Reference** | https://apidocs.nextroll.com/graphql-reporting-api/reference.html |
| **Examples** | https://apidocs.nextroll.com/graphql-reporting-api/examples.html |
| **SDL in this repo** | [`adroll-reporting.graphql`](adroll-reporting.graphql) — 107 types, 1,710 fields |

## Authentication

Same as every other NextRoll service: OAuth 2.0 (`Authorization: Bearer …`) or a
Personal Access Token (`Authorization: Token …`), **plus** the application's
client ID in the `apikey` query parameter on every request.

## Introspection is gated

Anonymous introspection is refused:

```
POST https://services.adroll.com/reporting/api/v1/query
{"query":"{__schema{queryType{name}}}"}

HTTP/1.1 401
{"errors":[{"code":"apiproxy:3","message":"Missing 'apikey' query parameter.
  Register for an API key at https://developers.nextroll.com/"}]}
```

The SDL in `adroll-reporting.graphql` is therefore **derived from the schema
reference NextRoll publishes**, field by field, not from an introspection
response. Nothing in it is invented.

## Error model

The Reporting API does not use standard GraphQL error handling. Responses carry a
top-level `has_errors` boolean and an `errors[]` array of `{id, msg}`; individual
objects carry their own `has_errors` plus `errors[]` of error **IDs** that
reference the top-level messages, and `has_errors` propagates up to the root. See
[`errors/adroll-error-codes.yml`](../errors/adroll-error-codes.yml).

## Provenance note

A previous round of this profile carried a hand-authored
`adroll-schema.graphql` that described itself as a *"conceptual schema derived
from the NextRoll REST API"*. It did not correspond to anything NextRoll
publishes and has been **deleted**. The file beside this one is the real schema.
