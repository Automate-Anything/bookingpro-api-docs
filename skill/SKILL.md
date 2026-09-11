---
name: booking-pro-api
description: >-
  Use this skill whenever you need to interact with the Booking Pro API - to
  read a service business's availability, catalog, contacts, bookings,
  gift cards, packages, or memberships, or to create/cancel/reschedule
  appointments, manage contacts, and check members in (kiosk, turnstile,
  biometric device) programmatically. Booking Pro is a SaaS for salons,
  barbershops, spas, gyms, and wellness studios. Trigger on: "book an
  appointment", "check availability", "create a contact", "list bookings",
  "check a member in", "Booking Pro API", "bp_sk_" keys.
license: Proprietary
metadata:
  homepage: https://developers.bookingpro.ai
  repository: https://github.com/Automate-Anything/bookingpro-api-docs
---

# Booking Pro API

You can operate a Booking Pro account programmatically over its REST API. This
one file tells you everything you need: how to authenticate, the conventions,
the full endpoint catalog, and where to find deeper detail. It is written to the
open [agentskills.io](https://agentskills.io) format, so it installs into Claude,
Cursor, Codex, GitHub Copilot, Gemini, and any agent that reads an `AGENTS.md`
or a skills directory. Install instructions are at the end.

## What Booking Pro is

A booking + point-of-sale platform for service businesses (salons, barbershops,
spas, gyms, wellness studios). Through this API you can read a business's
availability and catalog, manage its contacts, create and change appointments,
read its stored value (gift cards, packages, memberships), and run member
check-in for a door, kiosk, turnstile, or biometric device.

## Authentication

Every request needs an API key sent as a bearer token:

```
Authorization: Bearer bp_sk_live_xxxxxxxxxxxx
```

Keys are minted by an account owner/admin in the Booking Pro dashboard under
**Settings -> Developers**. A key is bound to ONE company and carries a set of
scopes plus a location scope. A key is a robot credential, not a logged-in user:
it can only do what its scopes allow. Never put a live key in client-side or
mobile code; treat it like a password. If a call returns `401` the key is
missing/invalid/revoked; `403` means the key lacks the required scope or the
target is outside its locations.

## Scopes

A call that needs a scope the key lacks returns `403 insufficient_scope`. Ask
the account owner to mint a key with exactly the scopes your integration needs.

| Tag | Scope |
| --- | --- |
| General | `(any valid key)` |
| Availability | `read:availability` |
| Catalog | `read:catalog` |
| Contacts | `read:contacts (read), write:contacts (create/update)` |
| Bookings | `read:bookings (read), write:bookings (create/cancel/reschedule)` |
| Gift Cards | `read:giftcards` |
| Packages | `read:packages` |
| Memberships | `read:memberships` |
| Members | `checkin:members` |

## Conventions

- Base URL: `https://api.bookingpro.ai/api/v1`
- All money is integer **cents** (e.g. `price_cents: 5000` is $50.00).
- All timestamps are ISO 8601 UTC.
- Responses are JSON. Errors use one envelope: `{ "error": { "type", "code", "message", "param?", "request_id" } }`. Branch on `type`, switch on `code`, quote `request_id` to support.
- Paginated list endpoints return `{ data: [...], next_cursor }`. Pass the `next_cursor` back as `?cursor=` for the next page; `null` means the end.
- Create endpoints accept an `Idempotency-Key` header; reuse the same key on a retry so you never create a duplicate.
- Rate limit is per key (default 100 req/min), with `X-RateLimit-*` headers and `Retry-After` on `429`.

## Full endpoint catalog

Every endpoint in `v1`. Each is fully specified (parameters, request body,
response) in `reference/endpoints.md` in this skill, and rendered with an
interactive playground at https://developers.bookingpro.ai.

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
| POST | `/members/resolve` | Resolve a member and check eligibility |
| POST | `/members/check-in` | Check a member in |
| GET | `/members/{id}/visits` | List a membership's check-ins |

## Common recipes

### Check availability then book
1. `GET /availability?location_id=...&service_id=...&date=2026-09-10` -> pick a `slots[].start`.
2. `POST /bookings` with `{ location_id, service_id, start, contact: { first_name, phone } }`
   (or an existing `contact_id`). Send an `Idempotency-Key`.
3. A `409` means the slot was taken between steps 1 and 2 - re-fetch availability.

### Create a contact
`POST /contacts` with `{ first_name, phone }`. Phone is normalized to E.164 and
de-duplicated: if the contact already exists it is returned, not duplicated.

### Check a member in (kiosk / turnstile / biometric device)
1. `POST /members/resolve` with `{ identifier, location_id }` -> returns the member and `valid_for_checkin` with a `reason`, recording nothing. A biometric device maps a face to the member's code or fob UID on your side, then sends that identifier.
2. If eligible, `POST /members/check-in` with the same `identifier` (or the `membership_id` + `contact_id`) and an `Idempotency-Key` so a double-scan logs one visit. A `422` means not allowed, with the reason in the message.

## Where to find more

- **Live interactive docs + playground**: https://developers.bookingpro.ai
- **Full endpoint detail**: `reference/endpoints.md` in this skill.
- **OpenAPI contract**: https://raw.githubusercontent.com/Automate-Anything/bookingpro-api-docs/main/openapi.yaml (or https://developers.bookingpro.ai/openapi.yaml)
- **Guides** (quickstart, auth, errors, pagination, idempotency, versioning, members, AI agents): the `docs/` folder of https://github.com/Automate-Anything/bookingpro-api-docs
- **LLM indexes**: https://developers.bookingpro.ai/llms.txt (compact) and https://developers.bookingpro.ai/llms-full.txt (every page concatenated for a context window). Append `.md` to any docs page URL for its Markdown.

## How to install this skill

This skill is a folder (`SKILL.md` + `reference/endpoints.md`). Give it to your
agent one of these ways:

- **Pull it from the public repo** (no download): point your agent at
  https://raw.githubusercontent.com/Automate-Anything/bookingpro-api-docs/main/skill/SKILL.md and tell it to read that file plus
  https://raw.githubusercontent.com/Automate-Anything/bookingpro-api-docs/main/skill/reference/endpoints.md. That is enough to use the API.
- **Download the bundle**: https://developers.bookingpro.ai/skill/booking-pro-api.zip , unzip it, then:
  - **Claude / Claude Code**: drop the `booking-pro-api` folder into your
    `.claude/skills/` directory (project or `~/.claude/skills/`). It loads by
    its `name` and `description`.
  - **Cursor**: put `SKILL.md`'s content into a `.cursor/rules/` rule, or add
    the folder to your project and reference it.
  - **Codex / GitHub Copilot / Gemini and other agents**: add this file's
    content to your `AGENTS.md` (or the agent's equivalent context/instructions
    file). The format is plain Markdown with YAML frontmatter, so any agent can
    read it.
- **Hand it to a chat model directly**: paste the contents of `SKILL.md` (and
  `reference/endpoints.md` for full detail) into the conversation, along with
  your `bp_sk_live_...` key, and ask it to call the API.

Once installed, the agent has everything it needs to authenticate and call the
Booking Pro API correctly.
