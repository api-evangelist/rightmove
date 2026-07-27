---
name: Withdraw or remove a commercial listing
description: Choose correctly between hiding a Rightmove commercial listing (reversible) and permanently deleting it (irreversible), and carry it out safely.
api: openapi/rightmove-commercial-listings-openapi.yml
operations:
  - sendCommercialPropertyDetails
  - removeCommercialPropertyDetails
  - getCommercialPropertyDetails
generated: '2026-07-26'
method: generated
---

# Withdraw or remove a commercial listing

There are two different actions here and they are not interchangeable. Pick deliberately — one is reversible and one is not.

## Decide first

| Situation | Action |
|---|---|
| Under offer, let agreed, sold subject to contract — still your instruction | **Update the status** with `sendCommercialPropertyDetails` |
| Temporarily off the market, may come back | **Unpublish** with `sendCommercialPropertyDetails` and `published: false` |
| Instruction lost, completed, or uploaded in error — gone for good | **Delete** with `removeCommercialPropertyDetails` |

Deletion is **permanent**. The property stops appearing on Rightmove, is not returned by future queries, and any associated listings are removed with it. There is no undelete. If an agent is running this, get explicit human confirmation before calling DELETE.

## Option A — change status (reversible)

Call **`sendCommercialPropertyDetails`** — `PUT /v2/property/commercial/{reference}` — resending the full payload with the new `status`. Permitted values, case-sensitive:

`AVAILABLE` · `SOLD_STC` · `SOLD_STCM` · `RESERVED` · `LET_AGREED` · `UNDER_OFFER`

Anything else (for example `SOLD_BY_US`) triggers an automatic `published: false` and the listing vanishes from search with no error returned. If a CRM status implies the property is no longer marketable, either stop sending it or map it onto one of the six values above.

## Option B — unpublish (reversible)

Same `PUT`, with `published: false`. Note the cascade:

- Unpublishing a **building** sets `published: false` on **all** of its spaces.
- Republishing the building does **not** bring the spaces back — each space must be sent again with `published: true`.

## Option C — delete (irreversible)

Call **`removeCommercialPropertyDetails`** — `DELETE /v2/property/commercial/{reference}` — with a body carrying:

- `agentId` — the branch/agent the property belongs to.
- `removalReason` — **required**; Rightmove will not accept a removal without one.

Responses: `200` removed · `404` no such property for this agent (do **not** retry — reconcile instead) · `401`/`403` credential or entitlement problem · `429` quota · `502`/`503` Rightmove-side, safe to retry because the call is keyed on the same client-supplied reference.

## Afterwards

Processing is asynchronous. Confirm with **`getCommercialPropertyDetails`** (`GET /v2/property/commercial/{reference}`, `Rightmove-Agent-ID` header required) or with the branch reconciliation, and keep `meta.traceId` / `properties.traceId` for support.

## If you deleted by mistake

There is no restore. Re-upload with `sendCommercialPropertyDetails` — the same `reference` will be treated as a new listing, and any Rightmove-side history associated with the old one is gone.
