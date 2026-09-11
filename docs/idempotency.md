# Idempotency

Make create requests safely retryable.

Network requests fail in ambiguous ways: your request may have succeeded even though you
never got the response. To make retries safe, send an **`Idempotency-Key`** header on
create requests.

```bash
curl -X POST https://api.bookingpro.ai/api/v1/bookings \
  -H "Authorization: Bearer bp_sk_live_..." \
  -H "Idempotency-Key: 8f1c2d3e-booking-42" \
  -H "Content-Type: application/json" \
  -d '{ "location_id": "...", "service_id": "...", "start": "2026-09-10T14:00:00Z", "contact_id": "..." }'
```

## How it works

- Generate a unique key per logical operation (a UUID works well).
- If you retry with the **same** key, you get the **original** result back instead of a
  second booking/contact.
- Use a **new** key for a genuinely new operation.

## Where it applies

`Idempotency-Key` is honored on:

- `POST /bookings`
- `POST /contacts`

It is safe to send on any create. Reads and updates are naturally idempotent and ignore
the header.
