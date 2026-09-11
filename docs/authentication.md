# Authentication

How API keys work, scopes, environments, and keeping keys safe.

The Booking Pro API authenticates every request with an API key sent as a bearer token:

```
Authorization: Bearer bp_sk_live_...
```

## Keys are per-company

A key is bound to exactly one company. Whatever company minted the key is the company
the request acts on, you never pass a company id, and you cannot reach another company's
data with your key. If a request body or query includes an id that belongs to a different
company, the API rejects it.

## Scopes

Each key carries a set of scopes. A request that needs a scope the key does not have
returns `403 insufficient_scope`. Grant only what an integration needs.

| Scope | Grants |
| --- | --- |
| `read:availability` | Read open slots |
| `read:catalog` | Read services and locations |
| `read:contacts` | Read contacts |
| `write:contacts` | Create and update contacts |
| `read:bookings` | Read bookings |
| `write:bookings` | Create, cancel, reschedule bookings |
| `read:giftcards` | Read gift cards |
| `read:packages` | Read package templates |
| `read:memberships` | Read membership plans and what a customer holds |
| `write:memberships` | Sell a membership to a contact |
| `checkin:members` | Resolve and check in members (kiosk, turnstile, biometric device) |

## Locations

A key is scoped to specific locations, or to **all** locations in the company (an
explicit choice at mint time). Reads and writes are always confined to the key's
locations. `/ping` shows which locations a key can act on.

## Environments

Keys are prefixed by environment:

- `bp_sk_live_...` acts on your real data.

A sandbox test environment (`bp_sk_test_...`) is on the roadmap; today only live keys
are issued.

## Keeping keys safe

- **Never** put a key in browser or mobile client code. Treat it like a password.
- Store keys in a secret manager or environment variables.
- Rotate periodically: create a new key, switch traffic, then revoke the old one.
- If a key leaks, revoke it immediately in **Settings -> Developers**. Revocation is
  instant.

## Errors

| Status | Meaning |
| --- | --- |
| `401 invalid_api_key` | Missing, malformed, or revoked key |
| `403 insufficient_scope` | Key lacks the required scope |
| `403` (location) | Target is outside the key's locations |

See [Errors](/docs/errors) for the full envelope.
