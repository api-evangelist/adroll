---
name: adroll-send-server-side-conversions
description: Send conversions and user events to AdRoll from your own servers with the Server-to-Server (S2S) Event API.
api: NextRoll Server-to-Server (S2S) Event API
base_url: https://srv.adroll.com
operations:
  - POST /api?advertisable=<ADVERTISABLE_EID>
generated: '2026-08-13'
method: generated
source: https://apidocs.nextroll.com/server-to-server-api/reference.html
---

# Send server-side conversions to AdRoll

The S2S Event API complements the browser pixel and MMP integrations for cases
where the conversion happens on your server. It is on a **different host**
(`srv.adroll.com`) with a **different credential** from the rest of the platform.

> NextRoll marks this API as under active development: "Although the API is
> generally stable, it may change. Event processing is not yet fully complete."

## Credential: this one is not self-service

You need a **Server Access Token (SAT)**. There is no dashboard page for it —
the docs say to contact your account manager, and NextRoll delivers it through a
one-time 1Password share link that expires after seven days. Plan for a human
step in your onboarding.

```
Authorization: Token MYTOKEN
```

## Endpoint

```
POST https://srv.adroll.com/api?advertisable=<ADVERTISABLE_EID>
POST https://srv.adroll.com/api?advertisable=<ADVERTISABLE_EID>&dry_run=true
```

`dry_run=true` validates and logs the payload **without** affecting audiences or
attribution. Use it for every first integration test — it is the only
non-destructive write path AdRoll publishes. Gzip request compression is
supported and recommended for batches.

## Payload

A JSON **array**; one request may carry many events.

```json
[{
  "advertisable_eid": "<ADVERTISABLE_EID>",
  "pixel_eid": "<PIXEL_EID>",
  "event_name": "purchase",
  "event_attributes": {},
  "conversion_value": "129.00",
  "currency": "USD",
  "page_location": "https://example.com/checkout/complete",
  "timestamp": "2026-08-13T10:04:00Z",
  "identifiers": {
    "adct": "click123",
    "first_party_cookie": "…",
    "email_sha256": "…",
    "device_id": "…",
    "user_id": "…"
  }
}]
```

**Identity rule:** every event must include at least one of `first_party_cookie`
or `adct`. More identifiers means a better match rate.

- `adct` — the click ID AdRoll appends to your landing page URL after an ad
  click. Capture it from the query string and persist it. If you use a
  third-party click tracker, make sure `adct` survives to the final landing page.
- `first_party_cookie` — read it from the pixel with
  `adroll.get_cookie(cb)`, or generate your own (valid one year) using the
  sample code at <https://github.com/AdRoll/server-to-server>.

## Event vocabulary

Use AdRoll's supported names — custom strings will not map to reporting.

- **B2C:** page view, home view, product search, add to cart, purchase
- **B2B / ABM:** high-value page, gated content, demo request, signup plan,
  signup trial, contact sales, live chat, form fill

The same vocabulary is available from the browser via `adroll.track(name, attrs)`
(see `components/adroll-components.yml`), so one taxonomy covers both paths —
send an event from exactly one of them to avoid double counting.

## Verify

Conversions land in the GraphQL Reporting API (`conversions`, `clickThroughs`,
`viewThroughs`, `revenue` on `Metric`). Remember reporting is UTC and not final
until 48 hours after the start of the current UTC day, so do not judge an
integration on same-hour numbers.

## Hygiene

- Hash emails (`email_sha256` / `email_md5`) rather than sending raw addresses.
- Send `timestamp` in ISO 8601 UTC.
- Batch events into one array rather than one request each — the daily call
  quota is small and this endpoint accepts gzip.
