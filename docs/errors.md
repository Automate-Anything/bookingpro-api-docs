# Errors

The error envelope, types, and how to handle failures.

Every error response uses the same JSON envelope and an appropriate HTTP status:

```json
{
  "error": {
    "type": "invalid_request",
    "code": "missing_field",
    "message": "first_name is required.",
    "param": "first_name",
    "request_id": "req_9f2b7c"
  }
}
```

- **`type`** groups the failure (see below). Branch on this.
- **`code`** is a stable, machine-readable string. Safe to switch on.
- **`message`** is human-readable. Do not parse it; it may change.
- **`param`** names the offending field, when applicable.
- **`request_id`** identifies this exact request. Quote it when contacting support.

## Types and statuses

| `type` | HTTP | When |
| --- | --- | --- |
| `invalid_request` | 400 / 422 | Malformed or failing-validation input |
| `authentication` | 401 | Missing or invalid API key |
| `permission` | 403 | Missing scope, or target outside the key's locations |
| `not_found` | 404 | The resource does not exist in your company |
| `conflict` | 409 | The requested time is no longer available |
| `rate_limit` | 429 | Too many requests, back off (see [Rate limits](./rate-limits.md)) |
| `server` | 500 / 503 | Something failed on our side, retry with backoff |

## Handling conflicts on booking

`POST /bookings` and `/bookings/{id}/reschedule` return `409 slot_unavailable` if the
slot was taken between your availability check and your write. The error carries
`alternatives`, up to 3 of the next open slots for the same service and staff member,
so you can offer the customer another time right away (or re-fetch
availability for more). Do not
blindly retry. A failed reschedule changes nothing: the original appointment stays.

## Locations in different currencies (`currency_mismatch`)

A business can run locations in different currencies (for example dollars in the US and
pesos in Mexico). Those locations are kept completely separate: nothing that carries money
or value crosses between them. A request that would (checking in a member whose plan is in
another currency than the location, using a package or credit bought in another currency,
and so on) answers `422` with `type: invalid_request` and `code: currency_mismatch`:

```json
{
  "error": {
    "type": "invalid_request",
    "code": "currency_mismatch",
    "message": "This membership is in US dollars and this location sells in Mexican pesos. Memberships can only be used at locations that use the same currency.",
    "request_id": "req_9f2b7c"
  }
}
```

The `message` is written for a person, so you can show it as is. **Do not retry it**: it is
a business rule, and the same request always gets the same answer. `POST /members/resolve`
reports the same case before you try, as `valid_for_checkin: false` with
`reason: currency_mismatch`.

## Retrying safely

Retry only `429` and `5xx`, with exponential backoff. Never retry a `4xx` such as
`currency_mismatch`. For creates, always send an
[`Idempotency-Key`](./idempotency.md) so a retry cannot create a duplicate.
