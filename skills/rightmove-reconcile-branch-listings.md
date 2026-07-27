---
name: Reconcile a branch's commercial listings
description: Page through everything Rightmove currently holds for a branch on the Commercial Listings API and diff it against the CRM's own stock, so nothing is orphaned or silently hidden.
api: openapi/rightmove-commercial-listings-openapi.yml
operations:
  - getCommercialPropertiesByBranch
  - getCommercialPropertyDetails
generated: '2026-07-26'
method: generated
---

# Reconcile a branch's commercial listings

Rightmove has **no webhooks and no event stream**. The only way to know what it holds is to ask. Run this reconciliation on a schedule — it is also the correct way to confirm that an asynchronous upload actually landed.

## Step 1 — token

Get a bearer token from `/oauth/token` (client credentials, ClientId/ClientKey from the Rightmove Data Feed Team) and send it as `Authorization: Bearer <ACCESS_TOKEN>`.

## Step 2 — page the branch

Call **`getCommercialPropertiesByBranch`** — `GET /v2/property/commercial/branch`.

- `id` (query, **required**) — the branch id.
- `page` (query, optional) — page number.
- `size` (query, optional) — page size.

Walk the pages until a page comes back short or empty. Collect every property reference Rightmove returns.

Keep `meta.requestTimestamp` / `meta.responseTimestamp` / `meta.traceId` from the `PropertyAction` envelope in your run log.

## Step 3 — diff

Compare the reference set against the CRM:

- **In CRM, not at Rightmove** — never landed, or was removed. Re-send with `sendCommercialPropertyDetails`.
- **At Rightmove, not in CRM** — orphaned stock still advertised publicly. Decide whether to withdraw it (see the withdraw skill).
- **In both** — pull the detail with **`getCommercialPropertyDetails`** (`GET /v2/property/commercial/{reference}`, with the required `Rightmove-Agent-ID` header) and compare status, pricing, sizing and media. Remember uploads are processed asynchronously, so allow a settling window before treating a difference as a failure.

## Step 4 — check for silent hiding

A property with a status outside `AVAILABLE`, `SOLD_STC`, `SOLD_STCM`, `RESERVED`, `LET_AGREED`, `UNDER_OFFER` is automatically flagged `published: false` and disappears from search **without an error**. Two traps to check on every run:

1. Anything with a non-conforming status — resend with a permitted status and `published: true`.
2. Spaces under a building that was ever unpublished — republishing the building does not republish its spaces; each space must be sent again with `published: true`.

## Rate limits

The quota window resets every 60 seconds and the quota value is not published (it varies per environment and endpoint). Pace the paging, back off exponentially on `429`, and do not depend on a `Retry-After` header — Rightmove does not commit to sending one.

## Errors

`401` re-authenticate · `403` the credentials are not entitled to that branch · `429` back off · `502`/`503` retry with backoff. Bodies are `ProblemDetail` JSON; quote `properties.traceId` to adfsupport@rightmove.co.uk.
