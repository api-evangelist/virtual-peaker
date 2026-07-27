---
name: Publish device telemetry to Virtual Peaker with HMAC signing
description: >-
  Stream signals, settings and energy intervals from an OEM cloud into the Virtual Peaker VPP
  platform, including how to compute the Authorization HMAC correctly and what the integration test
  actually checks.
api: openapi/virtual-peaker-gravity-connect-vpp-api-openapi.yml
also: openapi/virtual-peaker-gravity-connect-device-partner-api-openapi.yml
operations: [publishSignalSetting, readSignalSetting, updateSignalSetting, readDeviceDetails, readDeviceEnergyInterval]
generated: '2026-07-27'
method: generated
---

# Publish device telemetry

Base URL: `https://partner.virtualpeaker.io/v1` (development stage:
`https://partner-dev.virtualpeaker.io/v1`, which is the default in Virtual Peaker's published
Postman collection).

## Signing every request

1. Build the exact JSON body you will send.
2. `hmac = HMAC_SHA256(secret, rawBody).hex()` — Virtual Peaker publishes a Node.js reference:
   `crypto.createHmac('sha256', secret).update(body).digest('hex')`.
3. Send header `Authorization: Publish <hmac>`.

Which secret:

| publish | secret |
|---|---|
| `publishSignalSetting`, `publishDeviceCommand` | `DEVICE_PUBLISH_SECRET` (issued per device at subscription) |
| `publishCommand`, `publishDeviceEnrollment`, `publishDevicePartnerDrivenEnrollment` | `PROGRAM_PUBLISH_SECRET` (issued when the program is set up) |

Sign the bytes you actually transmit. Any re-serialisation between signing and sending breaks the
signature and you get a bare `401` with no explanation. The header is `Authorization` — spec 1.2.0
exists solely because integrators kept sending `Authentication`.

## Steps

1. **Publish signals and settings.** `publishSignalSetting`
   (`POST /publish/{PROGRAM_PUBLISH_KEY}/update`) with
   `{ uid, kind, signal: [{key, value, time}], setting: [{key, value, time}] }`. `value` is a
   string or a number; `time` is RFC 3339. Expect **202 Accepted**.
2. **Use the right key vocabulary.** Valid `key` values are device-type-specific — battery, TSTAT
   (thermostat), HWH (water heater), EVSE (spec 1.5.0), storage HVAC (spec 1.4.1). Reporting an
   unmodelled key is a `400`.
3. **Meet the cadence.** Integration testing requires either power data in **5-minute increments**
   or the `readDeviceEnergyInterval` endpoint on the device partner side
   (`GET /device/{DEVICE_UID}/energy`), whose `EnergyInterval` is `{ value (Wh), time (start of
   interval), duration (seconds) }`.
4. **Support the debugging reads.** Virtual Peaker's own Postman collection groups
   `readDeviceDetails` (`GET /device/{DEVICE_UID}`), `readSignalSetting` and `updateSignalSetting`
   (`/device/{DEVICE_UID}/{SIGNAL_OR_SETTING}/{DATA_KEY}`) as the debugging surface. Note this is
   the only place in Gravity Connect that returns a `404` — unknown `DATA_KEY`.
5. **Keep publishing during events.** Telemetry does not pause while a command is active; the
   active mode must match the commanded mode, which is how event performance is measured.

## Rules

- Max body **262,144 bytes** — batch signals per device, not per fleet.
- Retry only on 429/502/503/504 with exponential backoff; discard everything else.
- No rate limit is published and no `X-RateLimit-*` or `Retry-After` headers are documented.
- No idempotency key and no ordering guarantee: every point carries its own `time`, so make
  publishes replay-safe on `(uid, key, time)`.
