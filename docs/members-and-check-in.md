# Members and check-in

Resolve a member from a code, fob, or biometric device, and check them in.

If you run a facility (a gym, a studio, a club), you can wire your own hardware,
a kiosk, a turnstile, or a biometric face/fingerprint scanner, to Booking Pro so
that when someone arrives, you can look up their membership and let them in.

The flow is two steps: **resolve**, then **check in**.

## The identifier

A member is identified by one of:

- their **member code** (a short scannable code shown in the customer app and on
  their contact record), or
- an **RFID fob or card UID** (the value a tap reader emits).

A biometric device does not send a face to us. Instead, your device maps a face
(or fingerprint) to one of these identifiers on your side, then sends us the
identifier. We resolve it to the member.

## Step 1: Resolve and check eligibility

Call `POST /members/resolve` with the identifier and the location. It returns the
member and whether they are allowed in right now, without recording anything:

```bash
curl -X POST https://api.bookingpro.ai/api/v1/members/resolve \
  -H "Authorization: Bearer bp_sk_live_..." \
  -H "Content-Type: application/json" \
  -d '{ "identifier": "A1B2C3D4", "location_id": "loc_..." }'
```

```json
{
  "found": true,
  "membership_id": "mem_...",
  "contact_id": "con_...",
  "resolved_by": "code",
  "valid_for_checkin": true,
  "reason": "eligible",
  "plan_name": "Unlimited Monthly",
  "status": "active"
}
```

Use `valid_for_checkin` to decide whether to open the door. If it is `false`,
`reason` tells you why (for example `no_credits_remaining`,
`membership_not_active`, `location_not_covered`). Nothing is logged, so you can
call resolve as often as you like.

## Step 2: Check in

Once you decide to let them in, call `POST /members/check-in` to record the
visit. You can pass the same `identifier` (we resolve it again for you) or the
explicit `membership_id` + `contact_id` from step 1:

```bash
curl -X POST https://api.bookingpro.ai/api/v1/members/check-in \
  -H "Authorization: Bearer bp_sk_live_..." \
  -H "Idempotency-Key: door-3-2026-09-08T14:03:11Z" \
  -H "Content-Type: application/json" \
  -d '{ "identifier": "A1B2C3D4", "location_id": "loc_...", "method": "device" }'
```

```json
{ "checked_in": true, "visit_id": "vis_...", "remaining_credits": 11 }
```

Always send an **`Idempotency-Key`** (for example the door id + timestamp) so a
double-scan or a network retry never logs two visits. `remaining_credits` is
`null` for unlimited plans.

If the check-in is not allowed, you get a `422` with a clear reason in the
message (the same rules the dashboard enforces: active membership, the plan
allows facility access, the location is covered, cross-location rules, and
available credits).

## Scope

Both endpoints require the `checkin:members` scope. Mint a dedicated key for your
door/kiosk device with only that scope, and revoke it if the device is lost.

## Reading a member's history

`GET /members/{membership_id}/visits` returns recent check-ins (newest first),
so you can show attendance or reconcile your turnstile logs.
