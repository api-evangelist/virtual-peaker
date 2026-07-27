---
name: Dispatch and settle a demand response event over Gravity Connect
description: >-
  Group enrolled devices, send an individual or group command, track its state, handle opt-outs and
  cancellation, and report status back so measurement and verification stays accurate.
api: openapi/virtual-peaker-gravity-connect-device-partner-api-openapi.yml
also: openapi/virtual-peaker-gravity-connect-vpp-api-openapi.yml
operations: [createGroup, readGroup, updateGroup, manageGroupDevices, sendCommand, readCommandState, cancelCommand, commandOptOut, publishCommand, publishDeviceCommand, deleteGroup]
generated: '2026-07-27'
method: generated
---

# Dispatch a demand response event

The VPP calls the device partner's endpoints with an OAuth 2.0 client-credentials token
(scope `basic_partner_read_write`); the device partner reports back with HMAC-signed publishes.

## Steps

1. **Build the target set.** `createGroup` (`POST /group`) returns a `GroupDetails` with a partner-
   scoped `uid`. `deviceUids` is optional at creation (spec 1.4.1); add or remove members later
   with `manageGroupDevices` (`POST /group/{GROUP_ID}/devices`). Inspect with `readGroup`, rename
   or re-scope with `updateGroup`, tear down with `deleteGroup`.
2. **Send the command.** `sendCommand` (`POST /command`) carries the device-type-specific
   `VP_COMMAND_OBJECT` — e.g. a battery `{ mode: "DISCHARGE", action: "POWER", power: 4000 }`, or a
   thermostat/water-heater CTA mode. Virtual Peaker mints the `COMMAND_REFERENCE_ID`. Commands
   arrive **at most 60 seconds before the start time**, no matter how far ahead the program manager
   scheduled the event, and `startTime` may be in the past (spec 1.4.1).
3. **Report command status.** The device partner publishes `publishCommand`
   (`POST /publish/{PROGRAM_PUBLISH_KEY}/command`) with a `CommandState` — `PENDING` (scheduled,
   not started), `IN_PROGRESS`, then a terminal state — always echoing `refId`. Sign with
   `PROGRAM_PUBLISH_SECRET`.
4. **Report per-device status inside a group.** Use `publishDeviceCommand`
   (`POST /publish/{PROGRAM_PUBLISH_KEY}/command/device`) with both `refId` and the device `uid`.
   Sign with `DEVICE_PUBLISH_SECRET`.
5. **Handle opt-outs.** Two different paths, and mixing them is the classic integration bug:
   - Individual device event → the device partner publishes `publishDeviceCommand` with status
     `OPT_OUT`, the device `uid` and a timestamp.
   - Group command → use `commandOptOut`
     (`POST /command/{COMMAND_REFERENCE_ID}/opt-out`) and set the **group** command status to
     `OPT_OUT`.
6. **Poll or cancel.** `readCommandState` (`GET /command/{COMMAND_REFERENCE_ID}`) reads current
   state. `cancelCommand` (`DELETE /command/{COMMAND_REFERENCE_ID}`) may be issued at any point in
   the command's duration — the device partner must **immediately** return devices to normal
   operating modes.
7. **Keep modes coherent.** A device may not report `NORMAL` while it is participating in a `SHED`
   event (it may once it has opted out). Modes must change as the command begins and ends, and
   telemetry keeps flowing throughout — this is what makes the M&V settlement defensible.

## Rules

- A device partner must accept **individual and group commands simultaneously** for the same device.
- Command semantics are per device `kind` **and** per `type` (model): all devices of the same kind
  and model must support the same command set.
- Retry only on 429/502/503/504 with exponential backoff; discard other errors.
- Max body 262,144 bytes; all timestamps RFC 3339.
- No idempotency key exists — deduplicate on `refId` (+ `uid` for per-device status).

## Related

- `errors/virtual-peaker-problem-types.yml` — what 400/401/404 actually mean here.
- `asyncapi/virtual-peaker-gravity-connect-webhooks.yml` — the publish/event surface.
