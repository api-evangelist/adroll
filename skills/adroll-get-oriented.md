---
name: adroll-get-oriented
description: Authenticate against the NextRoll API for AdRoll and resolve the Advertisable EID every other call depends on.
api: NextRoll API for AdRoll
base_url: https://services.adroll.com
operations:
  - GET /api/v1/organization/get
  - GET /api/v1/organization/get_advertisables
  - GET /api/v1/organization/get_advertisables_paginated
  - GET /api/v1/advertisable/get
generated: '2026-08-13'
method: generated
source: https://apidocs.nextroll.com/guides/get-started.html
---

# Get oriented on the NextRoll API

Do this first. Almost every AdRoll call takes an **Advertisable EID**, and you
cannot guess one.

## 1. You need TWO credentials, not one

Every request carries both:

| Credential | Where | Example |
|---|---|---|
| User credential | `Authorization` header | `Authorization: Token MYTOKEN` (Personal Access Token) or `Authorization: Bearer {ACCESS_TOKEN}` (OAuth 2.0) |
| Application client ID | `apikey` **query parameter** | `?apikey=MYAPIKEY` |

The `apikey` is **always** in the URL query string — never in the body, even for
`POST`/`PUT`/`PATCH`. Omitting it fails before your token is ever checked:

```
HTTP/1.1 401
{"errors":[{"code":"apiproxy:3","message":"Missing 'apikey' query parameter."}]}
```

Register the application at <https://developers.nextroll.com/my-apps/new-app> to
get the client ID. Create a Personal Access Token at
<https://app.adroll.com/settings/personal-access-tokens>.

Set `Accept: application/json`. HTTPS only.

## 2. Resolve the Advertisable EID

```bash
curl -H 'Authorization: Token MYTOKEN' \
  'https://services.adroll.com/api/v1/organization/get_advertisables?apikey=MYAPIKEY'
```

For large organizations use `GET /api/v1/organization/get_advertisables_paginated`
instead — it is a *separate endpoint*, not a parameter on the one above.

## 3. Read the response envelope correctly

Success wraps the payload in `results`:

```json
{ "results": { "advertisable": "48F9EA2E5ACAEE24EB766F", "...": "..." } }
```

Failure **replaces** `results` with `errors`:

```json
{ "errors": [ { "message": "Please enter a name for your ad", "field": "name" },
              { "message": "No Advertisable found", "code": 16 } ] }
```

Each error carries a `message` plus either a numeric `code`
(1 INVALID, 2 MISMATCH, 4 NO_DEFAULT, 8 FAIL, 16 NOT_FOUND, 32 UNSET,
64 INCOMPLETE, 128 EXTERNAL, 256 DUPLICATE, 512 FORBIDDEN) or a `field` name.
Check for the `errors` key before touching `results`.

## 4. Object hierarchy — know where you are

```
Organization → Advertisable → Campaign → AdGroup → (Ads, Audiences)
```

Every object is addressed by an **EID**: an untyped alphanumeric string such as
`48F9EA2E5ACAEE24EB766F`. EIDs carry no type prefix, so keep track of which class
each one belongs to yourself.

## Rules that will bite you

- **There is no test mode.** Every credential addresses a live advertising
  account. Writes spend real money. See `sandbox/adroll-sandbox.yml`.
- **There is no idempotency key.** Retrying a failed `POST .../create` may create
  a duplicate. Read back before retrying a write.
- **The OAuth scope is `all`.** There is no read-only grant; a token you hold can
  do anything the user can do.
- **Quotas are small and the docs disagree** — one guide says 100 requests per
  service per day, another says 10,000 per day on the Basic tier. Exhaustion
  returns `429`, and **no rate-limit headers are published**, so budget your
  calls rather than reacting to a counter.
- Prefer one large GraphQL reporting query over many REST report calls; NextRoll
  says there is no size limit on GraphQL queries and it costs you one call.
