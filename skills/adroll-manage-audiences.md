---
name: adroll-manage-audiences
description: Create and maintain AdRoll CRM segments, ABM target account lists and ideal customer profiles through the Audience API.
api: NextRoll Audience API
base_url: https://services.adroll.com
operations:
  - GET /audience/v1/segments
  - POST /audience/v1/segments
  - GET /audience/v1/segments/(segment_id)
  - POST /audience/v1/segments/(segment_id)
  - DELETE /audience/v1/segments/(segment_id)
  - POST /audience/v1/segments/bulk
  - POST /audience/v1/segments_bulk/put
  - POST /audience/v1/segments/(segment_id)/reactivate
  - GET /audience/v1/target_accounts
  - POST /audience/v1/target_accounts
  - GET /audience/v1/target_accounts/(tal_eid)/tiers
  - POST /audience/v1/target_accounts/(tal_eid)/tiers/(ta_tier_eid)/items
  - GET /audience/v1/ideal_customer_profile
  - GET /audience/v1/ideal_customer_profile/all_scores
  - GET /user-lists/api/v1/userlists/segment
generated: '2026-08-13'
method: generated
source: https://apidocs.nextroll.com/audience-api/reference.html
---

# Manage AdRoll audiences

The Audience API is a **different service** from the CRUD API — different base
path (`/audience/v1`), different resource style (path parameters and JSON, not
RPC-style verbs in the path), same credentials.

## CRM segments

```bash
# list
curl -H 'Authorization: Token MYTOKEN' \
  'https://services.adroll.com/audience/v1/segments?apikey=MYAPIKEY&advertisable=ADVERTISABLE_EID'

# create
curl -X POST -H 'Authorization: Token MYTOKEN' -H 'Content-Type: application/json' \
  'https://services.adroll.com/audience/v1/segments?apikey=MYAPIKEY' -d '{...}'
```

- `POST /audience/v1/segments/bulk` and `POST /audience/v1/segments_bulk/put`
  handle large membership loads — use these rather than looping single writes,
  because the daily call quota is small.
- `DELETE /audience/v1/segments/(segment_id)` deactivates;
  `POST /audience/v1/segments/(segment_id)/reactivate` brings it back.
- `GET /audience/v1/segments/general_exclusions` returns account-level exclusions.

## Check size before you target

```bash
curl -H 'Authorization: Token MYTOKEN' \
  'https://services.adroll.com/user-lists/api/v1/userlists/segment?apikey=MYAPIKEY&segment=SEGMENT_EID'
```

The User Lists API reports audience sizes (`/segment`, `/segment/exact`,
`/segment/cdp_plus`, `/audience_preview`, plus `/advertisable`, `/campaign`-level
`/adgroup` and `/ad` variants). A segment too small to serve is the most common
reason a correctly-built campaign delivers nothing.

## AdRoll ABM: target account lists

```
GET    /audience/v1/target_accounts
POST   /audience/v1/target_accounts
GET    /audience/v1/target_accounts/(tal_eid)
GET    /audience/v1/target_accounts/(tal_eid)/tiers
POST   /audience/v1/target_accounts/(tal_eid)/tiers
POST   /audience/v1/target_accounts/(tal_eid)/tiers/(ta_tier_eid)/items
POST   /audience/v1/target_accounts/(tal_eid)/tiers/(ta_tier_eid)/items/filter
POST   /audience/v1/target_accounts/(tal_eid)/tiers/(ta_tier_eid)/items/delete
DELETE /audience/v1/target_accounts/(tal_eid)
```

Lists hold **tiers**, tiers hold **items** (accounts). Resolve company domains
with `GET /audience/v1/target_accounts/domains` and
`POST /audience/v1/target_accounts/domain_references`; link a TAL to a segment
with `PUT /audience/v1/segments/tal_references`.

## Ideal Customer Profile

`GET /audience/v1/ideal_customer_profile` lists ICPs,
`GET /audience/v1/ideal_customer_profile/all_scores` returns account scores, and
`POST /audience/v1/ideal_customer_profile/accounts` submits accounts for scoring.
This is the surface behind "prioritize high-intent accounts".

## Sharing audiences

`/audience/v1/sharing/invitation` and `/audience/v1/sharing/segment`
(GET/POST/DELETE) cover cross-account audience sharing. NextRoll flags this as
an **upcoming feature that is not available to everyone** — expect a 403 on
accounts without it rather than a clean capability check.

## Handling PII

CRM segments accept hashed identifiers. When sending identity to AdRoll, prefer
`email_sha256` / `email_md5` over raw `email` (both are accepted on the S2S event
envelope), and never log the raw values. AdRoll's own opt-out and privacy
obligations are documented at <https://www.nextroll.com/trust-center>.

## Retries

No idempotency key exists. On a timeout, re-`GET` the segment list and match on
name before re-`POST`ing, or you will create a duplicate audience.
