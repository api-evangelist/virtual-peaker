---
name: Enroll utility-commissioned devices with Gravity Connect pairing codes
description: >-
  Run the installer / program-manager enrollment paths — publish houses for commissioned
  installation, validate the pairing code, resolve the device's user, and complete either pairing
  code or device-partner-driven enrollment in bulk.
api: openapi/virtual-peaker-gravity-connect-device-partner-api-openapi.yml
also: openapi/virtual-peaker-gravity-connect-vpp-api-openapi.yml
operations: [publishHouseList, readDeviceUser, publishDeviceEnrollment, publishDevicePartnerDrivenEnrollment, modifySubscription]
generated: '2026-07-27'
method: generated
---

# Pairing-code and partner-driven enrollment

Use these flows where an installer or program manager, not the homeowner, drives enrollment.

## Pairing code format

`A1123455` = 2 alphanumeric characters (the program prefix) + 5 random numerics + 1 Luhn check
digit computed from those 5 numerics.

Validate before you accept it:

1. Check characters 1–2 against the program's pairing-code prefix.
2. Extract characters 3–7.
3. Compute the Luhn check digit over those 5 digits.
4. Compare with character 8.

Both sides may validate; the device partner validating first gives the installer immediate
feedback instead of a silent failure later.

## Utility-commissioned installation

1. **Publish the house list.** The VPP calls `publishHouseList` (`POST /houses`) on the device
   partner with the premises to be commissioned. `ServiceAddress.country` is **required** and is
   ISO 3166-1 alpha-2 (spec 1.3.3); US `state` is the 2-letter abbreviation. An optional array of
   devices may be included (spec 1.3.2).
2. **Resolve the account at install time.** `readDeviceUser` (`GET /device/{DEVICE_UID}/user`)
   returns `UserDetails` — `userId`, optional utility `accountNumber`, `name`, `email`,
   `deviceUids`, `serviceAddress` — so the device can be tied to the right utility customer.
3. **Publish the enrollment.** `publishDeviceEnrollment`
   (`POST /publish/{PROGRAM_PUBLISH_KEY}/device`) with `state: "enrolled"`, the `DeviceDetails`,
   `time`, and `pairingCode` — the pairing code is passed **only** in this flow.
4. **Subscribe.** The VPP calls `modifySubscription` (`POST /subscription`) to enable publishing
   and to hand over the `DEVICE_PUBLISH_SECRET`.

## Device-partner-driven / bulk enrollment

Use `publishDevicePartnerDrivenEnrollment`
(`POST /publish/{PROGRAM_PUBLISH_KEY}/enrollment`, added in spec 2.0.5) for in-app enrollment,
pre-enrollment from OEM-owned marketplaces, and bulk device enrollment. Body:
`{ devices: DeviceDetails[], site: UserDetails, time }` — one site, many devices, signed with the
`PROGRAM_PUBLISH_SECRET`.

## Rules

- One program = one utility (spec 1.1.0); `PROGRAM_PUBLISH_KEY` is unique per program **and**
  device partner.
- Max body 262,144 bytes — chunk bulk enrollments.
- Retry only on 429/502/503/504 with exponential backoff.
- No idempotency key: dedupe bulk enrollment on `device.uid`, and expect the platform to treat a
  repeat `enrolled` as a no-op rather than relying on it.
