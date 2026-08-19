---
name: adroll-launch-retargeting-campaign
description: Build an AdRoll retargeting campaign end to end — campaign, adgroup, ads, audiences — and leave it as a draft for human review.
api: NextRoll API for AdRoll (CRUD API)
base_url: https://services.adroll.com
operations:
  - POST /api/v1/campaign/create
  - POST /api/v1/adgroup/create
  - POST /api/v1/ad/create
  - POST /api/v1/adgroup/select_ads
  - POST /api/v1/adgroup/add_segments
  - GET /api/v1/advertisable/get_segments
  - GET /api/v1/campaign/get
  - PUT /api/v1/campaign/pause
  - PUT /api/v1/campaign/unpause
generated: '2026-08-13'
method: generated
source: https://apidocs.nextroll.com/crud-api/reference.html
---

# Launch an AdRoll retargeting campaign

⚠️ **There is no sandbox.** Every call below writes to a live advertising
account and can spend money. Leave the campaign in `draft` and hand it to a human
before it serves. AdRoll's own MCP server works this way — campaigns created by
agents are staged as drafts for review.

Prerequisite: the Advertisable EID (see `adroll-get-oriented`).

## 1. Pick the audiences

```bash
curl -H 'Authorization: Token MYTOKEN' \
  'https://services.adroll.com/api/v1/advertisable/get_segments?apikey=MYAPIKEY&advertisable=ADVERTISABLE_EID'
```

Audiences (a.k.a. Segments) are Conversion, CRM or URL based, and are populated
by the pixel. `GET /api/v1/pixel/get_segments` and `GET /api/v1/rule/get_segments`
show which pixel rules feed which segment.

## 2. Create the campaign

```bash
curl -X POST -H 'Authorization: Token MYTOKEN' \
  'https://services.adroll.com/api/v1/campaign/create?apikey=MYAPIKEY&advertisable=ADVERTISABLE_EID&name=Q3+Retargeting'
```

Parameters may go in the query string **or** in an
`application/x-www-form-urlencoded` / `multipart/form-data` body — except
`apikey`, which is always in the query string.

Campaign `status` will be one of: `draft`, `admin_review`, `approved`, `paused`,
`admin_paused`, `completed`, `cancelled`, `rejected`. A separate `is_active` flag
records deletion and is independent of `status`.

## 3. Create the adgroup — it is the join

```bash
curl -X POST -H 'Authorization: Token MYTOKEN' \
  'https://services.adroll.com/api/v1/adgroup/create?apikey=MYAPIKEY&campaign=CAMPAIGN_EID&name=Cart+Abandoners'
```

AdGroups join **Audiences** to **Ads**. That is their whole job.

## 4. Upload creative

```bash
curl -X POST -H 'Authorization: Token MYTOKEN' \
  -F 'ad_file=@banner-300x250.png' \
  -F 'name=Q3 Banner 300x250' \
  -F 'destination_url=https://example.com/sale' \
  'https://services.adroll.com/api/v1/ad/create?apikey=MYAPIKEY&advertisable=ADVERTISABLE_EID'
```

- `POST /api/v1/ad/clone` copies an existing ad.
- `POST /api/v1/ad/create_templated_web_ads` builds ads from a template.
- For dynamic creative, configure a product feed first
  (`PUT /api/v1/product_feeds/add_feed_config`, then
  `GET /api/v1/product_feeds/feed_status` and
  `GET /api/v1/product_feeds/match_rate`).

If the destination URL redirects, set the final URL in `display_url_override`…
except **for web campaigns that parameter is deprecated**. Check the FAQ before
relying on it.

## 5. Wire ads and audiences into the adgroup

```bash
curl -X POST -H 'Authorization: Token MYTOKEN' \
  'https://services.adroll.com/api/v1/adgroup/select_ads?apikey=MYAPIKEY&adgroup=ADGROUP_EID&ads=AD_EID_1,AD_EID_2'

curl -X POST -H 'Authorization: Token MYTOKEN' \
  'https://services.adroll.com/api/v1/adgroup/add_segments?apikey=MYAPIKEY&adgroup=ADGROUP_EID&segments=SEGMENT_EID_1'
```

Reverse them with `adgroup/deselect_ads` and `adgroup/remove_segments`.
Pause granularly with `adgroup/pause`, `adgroup/pause_ad`, `adgroup/pause_ads`
and their `unpause` twins; pause a whole campaign with
`PUT /api/v1/campaign/pause`.

## 6. Verify, then stop

```bash
curl -H 'Authorization: Token MYTOKEN' \
  'https://services.adroll.com/api/v1/campaign/get?apikey=MYAPIKEY&campaign=CAMPAIGN_EID'
```

Confirm the campaign, its adgroups, its selected ads and its segments read back
correctly — then **hand off to a human**. Approval flows through
`admin_review` before anything serves.

## Retry discipline

There is **no idempotency key** on this API. If a create call times out or
returns 5xx, do **not** blindly retry: call the matching `get` (e.g.
`GET /api/v1/advertisable/get_campaigns`) and check whether the object already
exists. A blind retry creates duplicates that spend budget.

Field validation errors come back as `{"errors":[{message, field, code}]}` with
HTTP 400 — fix the named `field` and resubmit.
