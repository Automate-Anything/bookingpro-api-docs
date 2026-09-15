# Booking Pro API

REST API for salons, barbershops, spas, and wellness studios to manage availability, catalog, contacts, bookings, and stored value (gift cards, packages, memberships) programmatically.

This repository is the canonical, public home of the Booking Pro API specification and documentation: the OpenAPI contract, the guide pages, and a ready-to-use AI Agent Skill.

## Live docs

- **Interactive reference and playground**: [developers.bookingpro.ai](https://developers.bookingpro.ai)
- **In-app authenticated version**: available inside your account at [app.bookingpro.ai](https://app.bookingpro.ai) under Settings -> Developers.
- **Product site**: [bookingpro.ai](https://bookingpro.ai)
- **Sign in / sign up**: [app.bookingpro.ai](https://app.bookingpro.ai)

## Base URL

```
https://api.bookingpro.ai/api/v1
```

The API is versioned in the path. Within `v1` we only make additive changes; see [docs/versioning.md](./docs/versioning.md).

## Authentication

Every request sends your API key as a bearer token:

```
Authorization: Bearer bp_sk_live_...
```

- Keys are minted in the dashboard under **Settings -> Developers** (owner or admin only).
- A key is bound to exactly one company, carries a set of scopes, and is scoped to specific locations (or all of them).
- Keys are revocable, and revocation is instant.
- Never commit a key or ship it in browser or mobile client code. Treat it like a password.

Full details: [docs/authentication.md](./docs/authentication.md).

## Conventions

- **Money is in integer cents.** `price_cents: 5000` means $50.00.
- **Times are ISO 8601 in UTC.**
- **Responses are JSON.**
- **Lists are cursor-paginated**: `{ "data": [...], "next_cursor": "..." }`. Pass the `next_cursor` back as `?cursor=` to get the next page; a `null` cursor means the end. See [docs/pagination.md](./docs/pagination.md).
- **Creates accept an `Idempotency-Key` header** so a retry never creates a duplicate. See [docs/idempotency.md](./docs/idempotency.md).
- **Errors use one envelope**: `{ "error": { "type", "code", "message", "param?", "request_id" } }`. Branch on `type`, switch on `code`, quote `request_id` to support. See [docs/errors.md](./docs/errors.md).
- **Rate limit** defaults to 100 requests per minute per key, with `X-RateLimit-*` headers and `Retry-After` on `429`. See [docs/rate-limits.md](./docs/rate-limits.md).

## Scopes

A request that needs a scope the key does not have returns `403 insufficient_scope`. Grant only what an integration needs.

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
| `read:classes` | Read the class timetable and rosters (gyms and studios) |
| `write:classes` | Book, cancel and check in class spots |

## Quickstart

Verify a key with `/ping`, which echoes back what the key can do:

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

Book an appointment: check availability, then create the booking. Send an `Idempotency-Key` so a retry never double-books:

```bash
# find open slots
curl "https://api.bookingpro.ai/api/v1/availability?location_id=a1c2....&service_id=svc_....&date=2026-09-10" \
  -H "Authorization: Bearer bp_sk_live_..."

# book one
curl -X POST https://api.bookingpro.ai/api/v1/bookings \
  -H "Authorization: Bearer bp_sk_live_..." \
  -H "Idempotency-Key: 8f1c-quickstart-1" \
  -H "Content-Type: application/json" \
  -d '{
    "location_id": "a1c2....",
    "service_id": "svc_....",
    "start": "2026-09-10T14:00:00Z",
    "contact": { "first_name": "Jordan", "phone": "+15551234567" }
  }'
```

A `409` on create or reschedule means the slot was taken between your availability check and your write. Re-fetch availability and offer another time. The full walkthrough is in [docs/quickstart.md](./docs/quickstart.md).

## Endpoints overview

Every endpoint in `v1`, derived from [`openapi.yaml`](./openapi.yaml):

| Method | Path | Summary |
| --- | --- | --- |
| GET | `/ping` | Verify a key |
| GET | `/availability` | List open slots |
| GET | `/services` | List services |
| GET | `/services/{id}` | Retrieve a service |
| GET | `/locations` | List locations |
| GET | `/contacts` | List contacts |
| POST | `/contacts` | Create a contact |
| GET | `/contacts/{id}` | Retrieve a contact |
| PATCH | `/contacts/{id}` | Update a contact |
| GET | `/bookings` | List bookings |
| POST | `/bookings` | Create a booking |
| GET | `/bookings/{id}` | Retrieve a booking |
| POST | `/bookings/{id}/cancel` | Cancel a booking |
| POST | `/bookings/{id}/reschedule` | Reschedule a booking |
| GET | `/gift-cards` | List gift cards |
| GET | `/gift-cards/{id}` | Retrieve a gift card |
| GET | `/packages` | List package templates |
| GET | `/memberships` | List membership plans |
| GET | `/contacts/{id}/memberships` | List a contact's memberships |
| POST | `/members/resolve` | Resolve a member (code, fob, or device) and check eligibility |
| POST | `/members/check-in` | Check a member in (records a visit) |
| GET | `/members/{id}/visits` | List a member's recent check-ins |
| GET | `/classes` | List classes (the timetable) |
| GET | `/classes/{id}` | Retrieve a class with its roster |
| POST | `/classes/{id}/book` | Book a contact into a class |
| POST | `/classes/{id}/cancel` | Cancel a contact's spot |
| POST | `/classes/{id}/check-in` | Check a booked contact in |

Each path is fully specified (parameters, request bodies, response schemas) in [`openapi.yaml`](./openapi.yaml), and rendered with an interactive playground at [developers.bookingpro.ai](https://developers.bookingpro.ai).

## Guides

Readable directly on GitHub in the [`docs/`](./docs) folder:

- [Quickstart](./docs/quickstart.md)
- [Authentication](./docs/authentication.md)
- [Members and check-in](./docs/members-and-check-in.md)
- [Classes](./docs/classes.md)
- [Errors](./docs/errors.md)
- [Rate limits](./docs/rate-limits.md)
- [Pagination](./docs/pagination.md)
- [Idempotency](./docs/idempotency.md)
- [Versioning](./docs/versioning.md)
- [Use with an AI agent](./docs/ai-agents.md)
- [Changelog](./docs/changelog.md)

## Use with an AI agent or LLM

The API is built to be driven by LLMs and AI agents.

- **AI Agent Skill** (works in Claude, Cursor, Codex, GitHub Copilot, Gemini, and any agent that reads a skills directory or an `AGENTS.md`): a ready-made, self-contained skill lives in the [`skill/`](./skill) folder of this repo: [`SKILL.md`](./skill/SKILL.md) plus [`reference/endpoints.md`](./skill/reference/endpoints.md). It is authored to the open [agentskills.io](https://agentskills.io) standard. `SKILL.md` carries the full scope table, the complete endpoint catalog, the conventions, the common recipes, and its own install instructions, so an agent can use the API from that one file.
  - **Point your agent straight at the repo** (no download): have it read `https://raw.githubusercontent.com/Automate-Anything/bookingpro-api-docs/main/skill/SKILL.md` and `.../skill/reference/endpoints.md`. Because this repo is public, any agent can fetch those and immediately know how to call the API.
  - **Or download the bundle**: [`https://developers.bookingpro.ai/skill/booking-pro-api.zip`](https://developers.bookingpro.ai/skill/booking-pro-api.zip), unzip, and drop the folder into your agent's skills directory (for Claude: `.claude/skills/`). A hosted copy of the file is at `https://developers.bookingpro.ai/skill/booking-pro-api/SKILL.md`.
- **llms.txt**: [`https://developers.bookingpro.ai/llms.txt`](https://developers.bookingpro.ai/llms.txt) is a compact index of the whole docs site, and `llms-full.txt` is every page concatenated for a context window.
- **Per-page Markdown**: append `.md` to any docs page URL, or click **Copy Markdown** at the top of a page, to fetch its Markdown.

More detail: [docs/ai-agents.md](./docs/ai-agents.md).

## Contributing and issues

This repository is the public specification and documentation for the Booking Pro API. It is the source developers integrate against. If you spot an error, an ambiguity, or a gap in the spec or the guides, please open an issue. Issues are welcome.

## License

The API specification and documentation in this repository may be used freely to build integrations with Booking Pro. The Booking Pro name and service are proprietary to Automate Anything LLC. See [LICENSE](./LICENSE).
