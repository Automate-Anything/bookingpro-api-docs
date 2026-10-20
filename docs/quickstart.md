# Quickstart

Get a key and make your first authenticated request.

## 1. Create an API key

In your Booking Pro dashboard, open the left sidebar, click **Settings**, then
**Developers** (you must be an owner or admin). Click **Create key**, choose the
scopes you need, and copy the key.

You will see the full key **once**. It looks like:

```
bp_sk_live_a1b2c3d4e5f6g7h8i9j0
```

Store it in a secret manager or an environment variable. If you lose it, revoke it and
create a new one.

## 2. Make your first request

Every request sends the key as a bearer token. Start with `/ping`, which just echoes
back what your key can do:

```bash
curl https://api.bookingpro.ai/api/v1/ping \
  -H "Authorization: Bearer bp_sk_live_..."
```

```json
{
  "company_id": "3b1e....",
  "environment": "live",
  "scopes": ["read:catalog", "read:bookings", "write:bookings"],
  "location_ids": ["a1c2...."],
  "request_id": "req_9f2b7c"
}
```

## 3. Do something real

List your locations, then your services at one of them:

```bash
curl https://api.bookingpro.ai/api/v1/locations \
  -H "Authorization: Bearer bp_sk_live_..."

curl "https://api.bookingpro.ai/api/v1/services?location_id=a1c2...." \
  -H "Authorization: Bearer bp_sk_live_..."
```

## 4. Book an appointment

Check availability, then book one of the slots it returned. Send an
`Idempotency-Key` so a retry never double-books:

```bash
# find open slots
curl "https://api.bookingpro.ai/api/v1/availability?location_id=a1c2....&service_id=svc_....&date=2026-09-10" \
  -H "Authorization: Bearer bp_sk_live_..."
```

```json
{
  "location_id": "a1c2....",
  "service_id": "svc_....",
  "slots": [
    { "start": "2026-09-10T14:00:00.000Z", "end": "2026-09-10T15:00:00.000Z", "technician_id": "t9d4...." }
  ]
}
```

Each slot names the staff member who is free then. Book the slot with that
`technician_id` (or leave `technician_id` out and the first free staff member is
assigned, the same person the slot named):

```bash
curl -X POST https://api.bookingpro.ai/api/v1/bookings \
  -H "Authorization: Bearer bp_sk_live_..." \
  -H "Idempotency-Key: 8f1c-quickstart-1" \
  -H "Content-Type: application/json" \
  -d '{
    "location_id": "a1c2....",
    "service_id": "svc_....",
    "technician_id": "t9d4....",
    "start": "2026-09-10T14:00:00.000Z",
    "contact": { "first_name": "Jordan", "phone": "+15551234567" }
  }'
```

If someone takes the time in between, you get `409 slot_unavailable` with
`error.alternatives`: up to 3 of the next open slots to offer instead.

That is the whole loop. Explore every endpoint in the
[API reference](https://developers.bookingpro.ai/docs/api-reference/general/ping), which has an interactive playground.
