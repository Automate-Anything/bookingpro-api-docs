# Changelog

API changes, additions, and deprecations.

## 2026-10-20: Booking a client who has no phone

- Fix: `POST /bookings` with a `contact_id` for a client who has an email but no phone
  number now keeps that client on the booking. Before, the booking was made without the
  client attached.

## 2026-10-20: Replaced gift cards, and what a cancel does with a deposit

Additive; no existing field changed.

- **Replaced gift cards are linked.** When a business replaces a lost or damaged card, the
  whole balance moves to a new card. `GET /gift-cards` and `GET /gift-cards/{id}` now
  return `replaced_by_gift_card_id` on the old card (it reads `voided` at balance 0; the
  customer kept their money on the new card) and `replaces_gift_card_id` on the new one. Both are `null` otherwise.
- **Cancel and deposits, documented.** `POST /bookings/{id}/cancel` turns anything the
  customer already paid toward the appointment into store credit with the business, and a
  late-cancellation fee from the business's policy is taken from that deposit first. No
  card is refunded automatically.

## 2026-10-16: Location limits everywhere, and a currency error code

- **Location-limited keys are now limited on every location record.** `GET /gift-cards/{id}`,
  `GET /contacts/{id}/memberships`, `GET /members/{id}/visits` and every webhook route now
  honor the key's locations (a record of another location answers `404`), and the
  documented `location_id` filter on `GET /gift-cards`, `GET /packages` and
  `GET /memberships` now narrows the list (it must be one of the key's locations).
- **New error code `currency_mismatch`** (`422`, `type: invalid_request`): a request that
  would carry money or value between locations that use different currencies. Do not
  retry it; the message says why. Class check-in and cancel used to answer
  `500 class_action_failed` for this case. See [Errors](./errors.md).
- **`POST /members/resolve` checks every check-in rule** (blackout window,
  home-location-only, the location's check-in blocks, per-member credits and the currency
  rule), so `valid_for_checkin: true` means the check-in will be accepted. New optional
  fields `message` and `block_reasons`.

## 2026-10: Built for chat and texting assistants

Additive; no existing field changed.

- **Find a client's appointments**: `GET /bookings` takes `phone` and `email` (matched
  the same way booking matches a contact, any stored phone format) and returns
  `contact_found` and `matched_contacts`, each with `client_status` (`new` or
  `returning`). New `exclude_cancelled` and `order` (`asc` / `desc`) filters.
- **Alternatives on a taken time**: `409 slot_unavailable` on create and reschedule
  now includes `error.alternatives`, up to 3 of the next open slots for the same
  service and staff member. A failed reschedule leaves the original untouched.
- **Availability**: the response includes the location's `timezone` and
  `next_available` (the earliest open slot, searched past the requested days when
  they have none). New `limit` parameter. Every slot carries `technician_name`.

## 2026-10: Scheduling fixes and new fields

Fixes (behavior now matches the public booking page):

- `GET /availability` returned slot times shifted by the location's UTC offset. Slot
  `start` / `end` are now the correct instants.
- "Any available": without `technician_id`, at a location where every appointment
  needs a staff member, availability is the union across everyone who performs the
  service, each slot names the staff member who is free (`technician_id`), and
  `POST /bookings` without `technician_id` assigns that person. At most 7 days per
  request in this mode (`422 days_too_large`).
- Availability, create and reschedule size the visit for the staff member doing it
  (their own duration, waiting period and cleanup time), and create and reschedule
  re-check the start against working and location hours and the booking rules.
- `POST /bookings/{id}/cancel` now cancels exactly like the dashboard (status,
  notifications, calendar, `booking.cancelled` webhook and automations).
- Rescheduling to another staff member re-sizes the visit for that person. A
  cancelled booking can no longer be rescheduled (`409 booking_cancelled`).
- `GET /services/{id}` and `POST /bookings` use the same catalog as `GET /services`
  (the location's price; deleted, inactive or not-offered services are not found).
  An unknown `service_id` on `GET /availability` is a `404` instead of a 30 minute guess.
- A malformed id (not a UUID) is a `422 invalid_id` / `invalid_field`, not a `500`.
- `GET /contacts?search=` finds a name written with or without accents ("jose" finds
  "José", "pena" finds "Peña"). Contacts that match the exact spelling come first, in
  the same order as before; accent-only matches are listed after them.
- Temporary read failures answer `503` (retry) instead of a partial result.

New fields:

- **Service**: `wait_minutes`, `finish_minutes`, `client_stays_in_room`.
- **Booking**: `status` is now always one of the documented values, mapped from the
  status's meaning (new values `checked_in`, `passed`); the business's own name is
  `status_name`. `staff_busy` lists when the staff member is actually busy. `slug`
  is documented.
- **Currency**: every priced object (`Location`, `Service`, gift cards, packages,
  membership plans) has a `currency` (uppercase ISO 4217), and webhook payment events
  (`payment.received`, `payment.refunded`) carry `data.currency` next to
  `amount_cents` and `tip_cents`. Additive; no existing field changed. Never assume USD.
- **Webhooks**: the delivery body is documented as `WebhookEvent`.

Spec corrections: `GET /services` is not paginated (it never was), and fields the API
never returned were removed from the spec (`Service.category`, `Location.slug`,
`Contact.location_id`, and the `location_id` filter on `GET /contacts`).

## 2026-09: Classes (gyms and studios)

- **Classes**: `GET /classes` (the timetable with booked / spots left), `GET /classes/{id}`
  (roster with what covers each spot, plus the waitlist), `POST /classes/{id}/book`,
  `POST /classes/{id}/cancel`, `POST /classes/{id}/check-in` (consumes the covering
  membership visit or package credit; honors the member check-in alert blocks).
- New scopes: `read:classes`, `write:classes`.

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
