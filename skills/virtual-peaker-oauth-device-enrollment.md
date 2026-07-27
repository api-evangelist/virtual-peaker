---
name: Enroll a DER device via Gravity Connect OAuth Device Discovery
description: >-
  Take a homeowner from consent to a subscribed, publishing device in a utility program using the
  preferred Gravity Connect enrollment flow — authorization-code OAuth on the device partner, then
  device discovery, subscription and the enrollment publish back to Virtual Peaker.
api: openapi/virtual-peaker-gravity-connect-device-partner-api-openapi.yml
also: openapi/virtual-peaker-gravity-connect-vpp-api-openapi.yml
operations: [readCurrentUser, readCurrentUserDevices, modifySubscription, publishDeviceEnrollment]
generated: '2026-07-27'
method: generated
---

# Enroll a device via OAuth Device Discovery

Gravity Connect is a **two-sided contract**. The device OEM ("Device Partner") implements one half;
Virtual Peaker (the VPP) implements the other. This skill covers the enrollment path Virtual Peaker
names as preferred.

## Before you start

- The device partner half is **OEM-hosted**. The published spec ships `https://example.com` as the
  `servers[]` placeholder — get the real base URL during onboarding.
- Credentials are **partner-only**, issued per utility program. Request them from
  `gravity-connect@virtual-peaker.com`.
- Two different auth models are in play — see `conventions/virtual-peaker-conventions.yml`:
  - VPP → Device Partner: **OAuth 2.0**. Client credentials with scope `basic_partner_read_write`
    for platform calls; authorization code with scope `user_read` for the consented calls below.
  - Device Partner → VPP: **HMAC-SHA256** of the raw request body, sent as
    `Authorization: Publish <hex>`.

## Steps

1. **Homeowner consents.** The device owner completes the VPP enrollment form, is redirected to the
   device partner's OAuth authorization page, signs in and grants access. The device partner
   completes the authorization-code exchange and calls back to the VPP with the token.
2. **Identify the user.** With the `user_read` token, call `readCurrentUser`
   (`GET /user`) to resolve the account — `userId`, optional utility `accountNumber`, `name`,
   `email`, `serviceAddress`.
3. **Discover devices.** Call `readCurrentUserDevices` (`GET /devices`) with the same token. Each
   `DeviceDetails` carries `uid`, `kind` (DeviceKindEnum), `type` (model), `serialNumber` and
   `isSubscribed`.
4. **Subscribe the devices you are enrolling.** The VPP calls `modifySubscription`
   (`POST /subscription`) on the device partner to enable publishing. This is the step that
   delivers the per-device `DEVICE_PUBLISH_SECRET`. Unsubscribing through the same operation is how
   the VPP unenrolls a device.
5. **Publish the enrollment.** The device partner POSTs `publishDeviceEnrollment`
   (`POST /publish/{PROGRAM_PUBLISH_KEY}/device`) with
   `{ state: "enrolled", device: DeviceDetails, time: <RFC 3339> }`, signed with the
   `PROGRAM_PUBLISH_SECRET`. Expect **202 Accepted**. Note: with OAuth Device Discovery, devices
   are assumed enrolled at discovery, so this confirms rather than initiates.
6. **Start telemetry.** Once subscribed, publish signals and settings continuously via
   `publishSignalSetting` — see the *publish-device-telemetry* skill.

## Rules

- `time` fields are RFC 3339 / ISO 8601; `country` on any address is **ISO 3166-1 alpha-2**
  (`US`, not `USA` — spec 1.3.3).
- Max request body is **262,144 bytes**.
- Retry only on **429, 502, 503, 504** with exponential backoff. Discard on every other error —
  Gravity Connect explicitly says so.
- There is **no idempotency key**. Re-publishing an enrollment is not deduplicated for you; key on
  `device.uid` + `time`.
- Errors are status-code-only: `400` and `401` have empty descriptions and there is no error
  schema. A `401` on the publish side almost always means a bad HMAC or the wrong header name —
  it is `Authorization`, not `Authentication`.

## Unenrollment

If the homeowner unenrolls in the OEM's app, publish `publishDeviceEnrollment` with
`state: "unenrolled"`. If a program manager unenrolls in the VPP, the VPP calls
`modifySubscription` to unsubscribe — treat that as an unenrollment.
