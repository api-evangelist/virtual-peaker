# Virtual Peaker (virtual-peaker)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Virtual Peaker is a Louisville, Kentucky software company selling grid-edge DERMS and virtual power plant software to United States and Canadian electric utilities — investor-owned utilities, municipal utilities, and rural electric cooperatives. Its platform is sold as three suites: Shift (grid-edge DERMS device control and demand response event dispatch), Relay (customer engagement, enrollment, and incentive processing), and Envision (demand forecasting). Virtual Peaker sits in the private DER-orchestration layer of the energy value chain, between device OEMs behind the meter and the utility back office — it is a vendor, not a utility, not a retailer, and not a data holder, so no consumer energy data right attaches to it. There is no Green Button / ESPI implementation here, no Consumer Data Right obligation, and no open market or grid data of any kind. What it does publish is its own API specification: Gravity Connect, an OpenAPI 3.0.0 contract (v2.0.6) that Virtual Peaker authored and openly published as a vendor-agnostic alternative to OpenADR and IEEE 2030.5 for onboarding and controlling DER devices. Gravity Connect is two-sided — the device OEM implements one half, the VPP platform implements the other — and both halves are readable anonymously as full Redoc API references. The honest posture: a real, downloadable, standards-ambitious API contract that is effectively undiscoverable (it is linked from nowhere on the marketing site) and whose credentials are partner-only, issued per utility program by emailing the Gravity Connect team. The commercial Shift API is named and sold on the marketing site but has no public documentation at all.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/virtual-peaker/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/virtual-peaker/refs/heads/main/apis.yml)

## Tags

- Energy
- United States
- Utilities
- Electricity
- Grid
- Demand Response
- DER
- DERMS
- Virtual Power Plant
- EV Charging
- Smart Thermostats
- Energy Storage

## Timestamps

- **Created:** 2026-07-27
- **Modified:** 2026-07-27

## APIs

### Gravity Connect API (Device Partner)

The Device Partner half of Virtual Peaker's Gravity Connect specification — the endpoints a device OEM must implement so a VPP or DERMS platform can discover, enroll, read, group, and command its behind-the-meter DER devices. Version 2.0.6, OpenAPI 3.0.0, 14 paths / 18 operations across Devices, Commands, Device Partner Driven Enrollment, OAuth Device Discovery, Pairing Code Device Discovery (end-user app and utility-commissioned installation), Group Management, and an Energy Interval endpoint. The published `servers` entry is the placeholder `https://example.com` because the OEM, not Virtual Peaker, hosts this surface. Secured with OAuth 2.0 — client credentials (scope `basic_partner_read_write`) for platform-to-partner calls, and authorization code (scope `user_read`) for homeowner-consented device discovery.

- **Human URL:** [https://assets.virtualpeaker.io/gravity-connect/device-partner-api.html](https://assets.virtualpeaker.io/gravity-connect/device-partner-api.html)

#### Tags

- DER
- Demand Response
- Device Control
- OEM
- OAuth

#### Properties

- [OpenAPI](openapi/virtual-peaker-gravity-connect-device-partner-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://assets.virtualpeaker.io/gravity-connect/device-partner-api.html)
- [Documentation](https://virtual-peaker.com/partners/device-partners/)
- [Blog](https://virtual-peaker.com/blog/gravity-connect-api-oem-derms-integrations/)

### Gravity Connect API (Virtual Peaker)

The VPP half of the Gravity Connect specification — the publishing endpoints Virtual Peaker hosts so an integrated device partner can stream device signals, settings, command status, and enrollment events back into the platform. Version 2.0.6, OpenAPI 3.0.0, 5 paths / 5 operations, all under the Publishing tag and all keyed on a per-program `PROGRAM_PUBLISH_KEY`. Base URL `https://partner.virtualpeaker.io/v1`, confirmed live (AWS API Gateway, anonymous request returns HTTP 403 `MissingAuthenticationTokenException`). Authentication is not OAuth on this side: requests are signed with an HMAC-SHA256 of the request body using a `PROGRAM_PUBLISH_SECRET` or `DEVICE_PUBLISH_SECRET` and sent as `Authorization: Publish <hmac>`.

- **Human URL:** [https://assets.virtualpeaker.io/gravity-connect/vp-api.html](https://assets.virtualpeaker.io/gravity-connect/vp-api.html)
- **Base URL:** `https://partner.virtualpeaker.io/v1`

#### Tags

- DER
- Demand Response
- Publishing
- Webhooks
- HMAC

#### Properties

- [OpenAPI](openapi/virtual-peaker-gravity-connect-vpp-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [API Reference](https://assets.virtualpeaker.io/gravity-connect/vp-api.html)
- [Documentation](https://virtual-peaker.com/apis/)
- [Blog](https://virtual-peaker.com/blog/gravity-connect-v-openadr/)

## Common

- [Website](https://virtual-peaker.com/)
- [Documentation](https://virtual-peaker.com/apis/)
- [API Reference](https://assets.virtualpeaker.io/gravity-connect/device-partner-api.html)
- [Blog](https://virtual-peaker.com/blog/)
- [Blog RSS](https://virtual-peaker.com/feed/)
- [Support Center](https://support.virtual-peaker.com/knowledge)
- [GitHub Organization](https://github.com/virtual-peaker)
- [LinkedIn](https://www.linkedin.com/company/virtual-peaker/)
- [Privacy Policy](https://virtual-peaker.com/privacy-policy/)
- [Terms of Service](https://virtual-peaker.com/terms-of-use/)
- [Contact Form](https://virtual-peaker.com/company/contact/)
- [llms.txt](https://virtual-peaker.com/llms.txt)

## Energy Data Posture

| Dimension | Finding |
| --- | --- |
| Home market | United States (secondary Canadian tenant at `utility.ca.virtualpeaker.io`) |
| Mandate regime | **none** — a DERMS/VPP software vendor is not a data holder under any energy consumer data right |
| Mandate status | **not-applicable** — no CDR energy obligation, no Ontario Green Button regulation, no US federal mandate exists, and no voluntary Green Button adoption was found |
| Data standard | Gravity Connect v2.0.6 (Virtual Peaker's own openly-published OpenAPI spec). No Green Button / ESPI, no CDR Consumer Data Standards, no OCPP/OCPI, no IEEE 2030.5, no IEC CIM. OpenADR and IEEE 2030.5 are named only as the incumbents Gravity Connect intends to displace |
| Consumer data API | **No** — no third-party access to a customer's metered usage or billing data; the nearest surface is a device-level Energy Interval endpoint the OEM implements under program enrollment |
| Open market data | **No** — publishes no grid, market, load, or emissions data; it ingests such data on behalf of utility customers |
| Access gate | **Partner-only** — the contract is anonymously readable, but credentials (`PROGRAM_PUBLISH_KEY`, `PROGRAM_PUBLISH_SECRET`, `DEVICE_PUBLISH_SECRET`) are issued per utility program after contacting `gravity-connect@virtual-peaker.com`. No signup, no sandbox, no pricing |
| Auth model | OAuth 2.0 client credentials + authorization code (platform → OEM); HMAC-SHA256 body signature `Authorization: Publish <hmac>` (OEM → platform) |

The full probe log, HTTP statuses, harvest provenance, and mandate reasoning are recorded in [review.yml](review.yml).
