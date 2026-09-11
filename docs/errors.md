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
slot was taken between your availability check and your write. Re-fetch availability and
offer the customer another time, do not blindly retry.

## Retrying safely

Retry only `429` and `5xx`, with exponential backoff. For creates, always send an
[`Idempotency-Key`](./idempotency.md) so a retry cannot create a duplicate.
