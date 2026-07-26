# Rightmove (rightmove)

Rightmove is the United Kingdom's largest residential property portal, operated by Rightmove Group Limited (Milton Keynes) and listed on the London Stock Exchange as Rightmove plc. It aggregates for-sale, to-let, new-homes, commercial and overseas listings supplied by member estate agents, letting agents and new-homes developers, and monetises the audience by charging those agents for advertising rather than by licensing data. The UK has no MLS, so Rightmove sits at the demand end of the value chain and its inbound feeds — not any cooperative database — are the machine-readable surface. Its API posture is publish-in, not read-out: the documented APIs let an agent's CRM or feed provider push listings into Rightmove, and there is no public API for reading listings, sold prices or valuations.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/rightmove/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/rightmove/refs/heads/main/apis.yml)

## Tags

- Real Estate
- United Kingdom
- Property Listings
- Property Portal
- PropTech
- Rentals
- Commercial Real Estate
- Data Feed

## Timestamps

- **Created:** 2026-07-26
- **Modified:** 2026-07-26

## APIs

### Rightmove Commercial Listings API

The Commercial Listings API allows commercial agents and feed providers to upload, update, retrieve and remove commercial property listings for display on the Rightmove website. It models a property as either a BUILDING or a SPACE, carries media (photos, floor plans, EPCs, EPC graphs, brochures, virtual tours) as publicly reachable URLs, and processes changes asynchronously. Residential listings are explicitly out of scope and must continue to go through the Real Time Data Feed API.

- **Human URL:** [https://api-docs.rightmove.co.uk/docs/property-feed-api-product/1/overview](https://api-docs.rightmove.co.uk/docs/property-feed-api-product/1/overview)
- **Base URL:** `https://api-services.rightmove.co.uk`

#### Tags

- Commercial Real Estate
- Property Listings
- Real Estate
- United Kingdom

#### Properties

- [OpenAPI](openapi/rightmove-commercial-listings-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://api-docs.rightmove.co.uk/docs/property-feed-api-product/1/overview)
- [API Reference](https://api-docs.rightmove.co.uk/apis)
- [Authentication](https://api-docs.rightmove.co.uk/authentication)
- [Getting Started](https://api-docs.rightmove.co.uk/get-started)
- [Sign Up](https://api-docs.rightmove.co.uk/accounts/create)
- [Terms of Service](https://api-docs.rightmove.co.uk/terms)
- [Support](mailto:adfsupport@rightmove.co.uk)

### Rightmove Real Time Data Feed API

The Real Time Data Feed (RTDF) is Rightmove's incremental HTTPS/JSON interface for UK sales, lettings and overseas sales listings, used by estate agency CRM and feed provider software rather than by end developers. The publicly downloadable specification (v1.4.1, November 2023) documents thirteen calls, including SendPropertyDetails, OverseasSendPropertyDetails, RemoveProperty, GetBranchPropertyList, AddPremiumListing, AddFeaturedProperty, RemoveFeaturedProperty, GetBranchPerformance, GetPropertyPerformance, GetBrandEmails, GetBranchEmails, GetPropertyEmails and ErrorCodes. Access is mutual-TLS, so neither the endpoints nor the referenced JSON Schemas are reachable anonymously.

- **Human URL:** [https://www.rightmove.co.uk/adf.html](https://www.rightmove.co.uk/adf.html)
- **Base URL:** `https://adfapi.rightmove.co.uk/v1`

#### Tags

- Property Listings
- Rentals
- Real Estate
- United Kingdom
- Data Feed

#### Properties

- [Documentation](https://www.rightmove.co.uk/adf.html)
- [Specification](https://media.rightmove.co.uk/ps/pdf/guides/adf/Rightmove_Real_Time_Datafeed_Specification.pdf) — Rightmove Real Time Data Feed API Web Services Specification v1.4.1
- [License](https://media.rightmove.co.uk/ps/pdf/guides/adf/RTDF_EULA.pdf) — RTDF End User Licence Agreement
- [Onboarding](https://media.rightmove.co.uk/ps/pdf/guides/adf/Provider_Contact_Form.pdf) — ADF Provider Contact Form
- [Support](mailto:adfsupport@rightmove.co.uk)

## Common Properties

- [Website](https://www.rightmove.co.uk/)
- [Developer Portal](https://api-docs.rightmove.co.uk/) — Rightmove APIs - Early adopters
- [Documentation](https://www.rightmove.co.uk/adf.html) — Rightmove Automated Datafeed (ADF) Specifications
- [Specification](https://media.rightmove.co.uk/ps/pdf/guides/ADF_V4n_specification.pdf) — Rightmove Automated Data Feed v4.0n (New Homes bulk XML file feed, not an HTTP API)
- [License](https://media.rightmove.co.uk/ps/pdf/guides/adf/RTDF_EULA.pdf) — RTDF End User Licence Agreement
- [Terms of Service](https://www.rightmove.co.uk/c/terms-of-use/)
- [Privacy Policy](https://www.rightmove.co.uk/c/privacy-policy/)
- [GitHub Organization](https://github.com/rightmove)
- [LinkedIn](https://www.linkedin.com/company/rightmove)
- [Investor Relations](https://plc.rightmove.co.uk/)
- [Blog](https://www.rightmove.co.uk/news/)
- [Support](mailto:adfsupport@rightmove.co.uk)

## RESO Posture

**No RESO reference found.** `reso_certified: false`

No RESO Web API certification, no RESO Data Dictionary certification at any version, no OData service root or `$metadata` document, and no Universal Property Identifier appears anywhere in Rightmove's surface. Rightmove uses its own proprietary schema — the harvested OpenAPI declares 50 Rightmove-designed component schemas with British transaction states (`AVAILABLE`, `SOLD_STC`, `SOLD_STCM`, `RESERVED`, `LET_AGREED`, `UNDER_OFFER`) that have no RESO Data Dictionary equivalent, and the RTDF uses Rightmove field names (`Network_ID`, `Branch_ID`, `Agent_Ref`, `Channel`). This is a true absence rather than a probing gap: RESO is a North American standard mandated by NAR for US MLSs, and the UK has no MLS and no RESO-equivalent industry mandate.

## Access Gate

**`application-approval`.** Every documented route to credentials goes through the Rightmove Data Feed Team at `adfsupport@rightmove.co.uk`, for both the test and production environments. A developer must also accept the RTDF End User Licence Agreement — a binding agreement with Rightmove Group Limited (company number 03997679) — submit the ADF Provider Contact Form, and complete a supervised testing cycle in which Rightmove issues a keystore by email and its password by SMS. In practice a Rightmove membership is required too: RTDF calls carry a Rightmove-issued `Network_ID` and `Branch_ID`, and the EULA distinguishes a "Member" from a non-Member acting as a Member's data processor.

The portal's Get Started page is stock Apigee boilerplate (sign in, register an app, read the keys) and the API product carries `approvalType: auto`, so it *looks* self-serve — but the API's own overview and authentication pages both state that working credentials are issued by the Data Feed Team. The portal is also explicitly pre-production: site id `rightmove-pd-preprod-apigee-commercialpoc`, sole category "Beta users", product description "PREPROD API Product giving agents access to manage properties via their feed providers/CRMs".

## Open Data

**None.** Rightmove publishes no open, unlicensed, publicly callable dataset. The Rightmove House Price Index is editorial and PDF, not an API, and `robots.txt` disallows `/api/*`. The genuinely open UK property data — HM Land Registry Price Paid, Ordnance Survey — belongs to the public sector, not to Rightmove.

## Auth Model

- **Commercial Listings API:** OAuth2 client credentials. `ClientId`/`ClientKey` issued by Rightmove, exchanged at `/oauth/token`, used as `Authorization: Bearer <ACCESS_TOKEN>`. Scopes are not implemented. The published spec's `securitySchemes` declares an `implicit` flow against `/oauth/token` while the prose says `client_credentials` — recorded verbatim, not corrected.
- **Real Time Data Feed API:** Mutual TLS. Rightmove supplies a keystore containing a private key and an X.509 certificate by email, with the password by SMS. TLS 1.2 required.
- **OpenID Connect:** none. `/.well-known/openid-configuration` returns a branded soft-404 HTML page.

## Webhooks, Events, SDKs, Postman

Absence is the finding. No webhooks or callbacks (the RTDF is poll-based, reconciled via `GetBranchPropertyList`), no event stream or AsyncAPI, no GraphQL, no gRPC, no Rightmove-published SDKs, no Postman collection, no public status page and no `security.txt`.

## Harvested Artifacts

| File | Source | Fetched | Status | Format |
|---|---|---|---|---|
| `openapi/rightmove-commercial-listings-openapi.yml` | `api-docs.rightmove.co.uk/portals/api/sites/rightmove-pd-preprod-apigee-commercialpoc/liveportal/apis/property-feed-api-product/download_spec` | 2026-07-26 | 200 | OpenAPI 3.0.1 (YAML), 75,326 bytes |

Full probe log, including every soft-404 and every 403, is in [`review.yml`](review.yml).
