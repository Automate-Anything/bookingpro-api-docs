# Changelog

API changes, additions, and deprecations.

## 2026-09: Members and check-in

- **Members**: `POST /members/resolve` (map a member code or fob UID to a member and
  check eligibility, for kiosks, turnstiles, and biometric devices),
  `POST /members/check-in` (record a facility check-in), `GET /members/{id}/visits`.
- **Memberships**: `GET /contacts/{id}/memberships` (what a customer holds, with status
  and remaining credits).
- New scopes: `write:memberships`, `checkin:members`.

## 2026-09: v1 launch

The first public version of the Booking Pro API:

- **General**: `GET /ping`.
- **Availability**: `GET /availability`.
- **Catalog**: `GET /services`, `GET /services/{id}`, `GET /locations`.
- **Contacts**: `GET /contacts`, `GET /contacts/{id}`, `POST /contacts`, `PATCH /contacts/{id}`.
- **Bookings**: `GET /bookings`, `GET /bookings/{id}`, `POST /bookings`,
  `POST /bookings/{id}/cancel`, `POST /bookings/{id}/reschedule`.
- **Stored value (read)**: gift cards, package templates, membership plans.
- Per-key API keys with scopes, cursor pagination, idempotency keys, and a published
  Claude Agent Skill.
