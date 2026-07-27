---
name: Publish or update a commercial property listing
description: Authenticate against the Rightmove Commercial Listings API and upsert a commercial building — with or without individual spaces — keyed on the agent's own reference, then confirm it landed.
api: openapi/rightmove-commercial-listings-openapi.yml
operations:
  - sendCommercialPropertyDetails
  - getCommercialPropertyDetails
generated: '2026-07-26'
method: generated
---

# Publish or update a commercial property listing

Use this skill to get a commercial building onto Rightmove, or to update one that is already there. **Residential stock does not go through this API** — it goes through the Real Time Data Feed, which is a different interface behind mutual TLS.

## Before you start

- You need a ClientId and ClientKey issued by the Rightmove Data Feed Team (`adfsupport@rightmove.co.uk`). Test and production credentials are different and both are issued by hand — there is no self-serve key.
- Pick the environment: `https://api-services.adftest.rightmove.com` (staging, never appears on the live site) or `https://api-services.rightmove.co.uk` (production).
- You need the branch's `agentId` and the agent's own `reference` for the property.

## Step 1 — get a token

POST to `/oauth/token` on the chosen host with the client-credentials grant, sending the base64-encoded ClientId/ClientKey. Use the returned token as `Authorization: Bearer <ACCESS_TOKEN>` on every call.

The OpenAPI declares this scheme as an `implicit` flow with an empty scopes map — that is a defect in the published document. The real grant is `client_credentials`, and scopes are not implemented. Do not attempt a browser redirect.

## Step 2 — decide the model

- Marketing the whole building only → body schema `CommercialPropertyBuildingOnly`.
- Marketing individual floors/units → body schema `CommercialPropertyWithSpaces`, up to **50** spaces.

Rules that fail the request if you get them wrong:

- The `reference` in the URL is the building's. When you send spaces, each space needs its **own unique** `reference`, different from the building's.
- `BuildingPricing` is mandatory at building level. `SpaceSizing` (`size` + `unit`) is mandatory for every space; `SpacePricing` is optional.
- `status` must be one of `AVAILABLE`, `SOLD_STC`, `SOLD_STCM`, `RESERVED`, `LET_AGREED`, `UNDER_OFFER`. Anything else silently flips `published` to false and hides the listing.
- Media URLs must be publicly reachable; brochure URLs must end in `.pdf`; asset `description` is capped at 200 characters. Serve an `ETag` — Rightmove issues a HEAD before downloading and skips unchanged assets.
- Do not send `constructionType` (building) or `model` (space); both were deprecated in v2.1.0.

## Step 3 — upsert

Call **`sendCommercialPropertyDetails`** — `PUT /v2/property/commercial/{reference}`.

- A reference Rightmove has not seen → **201 Created**.
- A reference it already holds → **200 OK**, the listing is updated.

This is the idempotency contract: the client-supplied `reference` is the key, so a retry after a timeout, a `429`, a `502` or a `503` cannot create a duplicate. There is no `Idempotency-Key` header — do not invent one.

Read `data.links.self.building` and `data.links.display.*` from the `PropertySaveAction` response, and keep `meta.traceId` in your logs; it is what support asks for.

## Step 4 — confirm

Processing is **asynchronous**. A 2xx means Rightmove accepted the payload, not that it is live on the site. Call **`getCommercialPropertyDetails`** — `GET /v2/property/commercial/{reference}` with the required `Rightmove-Agent-ID` header — to read back what Rightmove now holds.

## Errors

| Status | What it means | What to do |
|---|---|---|
| 400 | Validation failed | Read `properties.validationError` for the field and message, fix, resend the same reference |
| 401 | Token missing/expired | Re-request a token from `/oauth/token` |
| 403 | Not entitled to this branch or property | Do not retry; check entitlement with adfsupport@rightmove.co.uk |
| 429 | Quota exceeded | Back off exponentially; the window resets every 60s and the quota value is not published |
| 502 / 503 | Rightmove-side failure | Retry with backoff — the PUT is idempotent |

Error bodies are `ProblemDetail` (`type`, `title`, `status`, `detail`, `instance`, `properties`) served as `application/json`.

## Changing a building into spaces

Keep the building `reference` the same, remove the building-level model object, and add the space objects — each with its own unique reference. The reverse (SPACE back to BUILDING) is **not supported**: you must `removeCommercialPropertyDetails` and re-upload.

## Visibility

Unpublishing a building (`published: false`) cascades to every space. Republishing the building does **not** republish the spaces — each space must be sent again with `published: true`.
