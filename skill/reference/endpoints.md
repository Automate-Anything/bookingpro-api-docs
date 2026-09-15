# Booking Pro API - Endpoint Catalog

Base URL: `https://api.bookingpro.ai/api/v1`
Auth: `Authorization: Bearer bp_sk_live_...` on every request.
All money is in integer cents. All times are ISO 8601 (UTC).

## Scopes

Each key carries a set of scopes chosen when it was minted. A call that needs a scope the key lacks returns `403 insufficient_scope`.

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

---

## GET /ping

Scope: `(any valid key)`

**Verify a key.** Returns the identity a key resolves to. Use it to confirm a key works and to see which company, scopes, environment, and locations it is bound to.

Success: `200`

---

## GET /availability

Scope: `read:availability`

**List open slots.** Returns bookable start times for a service at a location on a given date (or date range). Honors the location's business hours, staff schedules, existing bookings, and booking rules, exactly like the public booking page.

Parameters:
- `location_id` (query) (required)
- `service_id` (query) (required)
- `date` (query) (required) - Start date (YYYY-MM-DD), interpreted in the location's timezone.
- `days` (query) - Number of days from `date` to include (1-31). Defaults to 1.
- `technician_id` (query) - Restrict to a specific staff member.

Success: `200`

---

## GET /services

Scope: `read:catalog`

**List services.** The bookable services offered, scoped to the key's locations.

Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.
- `limit` (query)
- `cursor` (query) - The `next_cursor` from a previous page.

Success: `200`

---

## GET /services/{id}

Scope: `read:catalog`

**Retrieve a service.** 
Parameters:
- `id` (path) (required)

Success: `200`

---

## GET /locations

Scope: `read:catalog`

**List locations.** The locations this key may act on.

Success: `200`

---

## GET /contacts

Scope: `read:contacts (read), write:contacts (create/update)`

**List contacts.** 
Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.
- `search` (query) - Fuzzy match on name, phone, or email.
- `limit` (query)
- `cursor` (query) - The `next_cursor` from a previous page.

Success: `200`

---

## POST /contacts

Scope: `read:contacts (read), write:contacts (create/update)`

**Create a contact.** Creates a customer record. Phone numbers are normalized to E.164. If a contact with the same phone already exists at the location, the existing record is returned (idempotent on phone) rather than duplicated.

Parameters:
- `Idempotency-Key` (header) - A unique key you generate per logical create. Retrying with the same key returns the original result instead of creating a duplicate.


Request body:ContactCreate

Success: `201`

---

## GET /contacts/{id}

Scope: `read:contacts (read), write:contacts (create/update)`

**Retrieve a contact.** 
Parameters:
- `id` (path) (required)

Success: `200`

---

## PATCH /contacts/{id}

Scope: `read:contacts (read), write:contacts (create/update)`

**Update a contact.** 
Parameters:
- `id` (path) (required)

Request body:ContactUpdate

Success: `200`

---

## GET /bookings

Scope: `read:bookings (read), write:bookings (create/cancel/reschedule)`

**List bookings.** 
Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.
- `from` (query) - Include bookings starting on/after this date-time.
- `to` (query) - Include bookings starting before this date-time.
- `contact_id` (query)
- `limit` (query)
- `cursor` (query) - The `next_cursor` from a previous page.

Success: `200`

---

## POST /bookings

Scope: `read:bookings (read), write:bookings (create/cancel/reschedule)`

**Create a booking.** Books an appointment. Enforces the location's booking rules and hard-blocks any time conflict (staff or room), exactly like the public booking page. A conflicting request returns `409`.

Parameters:
- `Idempotency-Key` (header) - A unique key you generate per logical create. Retrying with the same key returns the original result instead of creating a duplicate.


Request body:BookingCreate

Success: `201`

---

## GET /bookings/{id}

Scope: `read:bookings (read), write:bookings (create/cancel/reschedule)`

**Retrieve a booking.** 
Parameters:
- `id` (path) (required)

Success: `200`

---

## POST /bookings/{id}/cancel

Scope: `read:bookings (read), write:bookings (create/cancel/reschedule)`

**Cancel a booking.** 
Parameters:
- `id` (path) (required)

Request body:
  - `reason`: string - Optional cancellation reason (stored on the booking).

Success: `200`

---

## POST /bookings/{id}/reschedule

Scope: `read:bookings (read), write:bookings (create/cancel/reschedule)`

**Reschedule a booking.** Moves a booking to a new start time. Same conflict rules as create.

Parameters:
- `id` (path) (required)

Request body:
  - `start`: string (required)
  - `technician_id`: string

Success: `200`

---

## GET /gift-cards

Scope: `read:giftcards`

**List gift cards.** 
Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.
- `limit` (query)
- `cursor` (query) - The `next_cursor` from a previous page.

Success: `200`

---

## GET /gift-cards/{id}

Scope: `read:giftcards`

**Retrieve a gift card.** 
Parameters:
- `id` (path) (required)

Success: `200`

---

## GET /packages

Scope: `read:packages`

**List package templates.** 
Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.

Success: `200`

---

## GET /memberships

Scope: `read:memberships`

**List membership plans.** 
Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.

Success: `200`

---

## GET /contacts/{id}/memberships

Scope: `read:memberships`

**List a contact's memberships.** The memberships a contact holds (as primary member or family member), with status and remaining credits.

Parameters:
- `id` (path) (required)

Success: `200`

---

## POST /members/resolve

Scope: `checkin:members`

**Resolve a member and check eligibility.** Given a credential (a member code or an RFID fob/card UID) and a location, returns the matching membership and whether the member is currently eligible to check in. Use this from a kiosk, turnstile, or a biometric device (map a face/fingerprint to the member's code, then call this) to decide whether to let someone in, without recording a visit.

Request body:
  - `identifier`: string (required) - The member's code or fob/card UID.
  - `location_id`: string (required)

Success: `200`

---

## POST /members/check-in

Scope: `checkin:members`

**Check a member in.** Records a facility check-in for a member. Provide either an `identifier` (member code or fob UID, which is resolved for you) or an explicit `membership_id` + `contact_id`. Enforces the same rules as the dashboard: membership active, plan allows facility access, the location is covered, cross-location rules, and available credits. Send an `Idempotency-Key` so a retry or a double-scan does not log two visits.

Parameters:
- `Idempotency-Key` (header) - A unique key you generate per logical create. Retrying with the same key returns the original result instead of creating a duplicate.


Request body:
  - `location_id`: string (required)
  - `identifier`: string - Member code or fob UID (resolve for me).
  - `membership_id`: string
  - `contact_id`: string
  - `method`: string - How the member was identified.

Success: `201`

---

## GET /members/{id}/visits

Scope: `checkin:members`

**List a membership's check-ins.** 
Parameters:
- `id` (path) (required)
- `limit` (query)

Success: `200`

---

## GET /classes

**List classes (the timetable).** Group classes scheduled in a window (default: the next 7 days, at most 31). Each row carries how many spots are booked and how many are left. Only gyms and studios run classes; other businesses get an empty list.

Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.
- `from` (query) - Window start (ISO 8601). Defaults to now.
- `to` (query) - Window end (ISO 8601). Defaults to from + 7 days; at most 31 days after from.
- `service_id` (query) - Only classes of this service.
- `limit` (query)
- `cursor` (query) - The `next_cursor` from a previous page.

Success: `200`

---

## GET /classes/{id}

**Retrieve a class with its roster.** 
Parameters:
- `id` (path) (required)

Success: `200`

---

## POST /classes/{id}/book

**Book a contact into a class.** Reserves a spot on the roster. Nothing is charged: the spot records what will cover it at check-in (`membership`, `package`, `free_class`) or `none`. The class rules apply: capacity (409 `class_full`), signup cutoff / max per day / first-timers only (422 `booking_rule`), and, when the service requires a membership or package, entitlement (422 `entitlement_required`) unless `allow_without_entitlement` is true (a staff-style booking the desk charges at checkout). Send an `Idempotency-Key`.

Parameters:
- `id` (path) (required)
- `Idempotency-Key` (header) - A unique key you generate per logical create. Retrying with the same key returns the original result instead of creating a duplicate.


Request body:
  - `contact_id`: string (required)
  - `allow_without_entitlement`: boolean - Book even when no membership or package covers the class (charge at checkout).

Success: `201`

---

## POST /classes/{id}/cancel

**Cancel a contact's spot.** Releases the contact's spot. Inside the location's cancellation window it is a late cancel (`late: true`) that still consumes the covering credit unless `excuse` is true. A freed spot is offered to the waitlist automatically.

Parameters:
- `id` (path) (required)

Request body:
  - `contact_id`: string (required)
  - `reason`: string
  - `excuse`: boolean - Return the credit even on a late cancel.

Success: `200`

---

## POST /classes/{id}/check-in

**Check a booked contact in.** Marks the contact's spot checked in and consumes what covers it (a membership visit or a package credit). Honors the location's member check-in alert blocks (422 `checkin_blocked` with `reasons`; a manager overrides at the desk). Send an `Idempotency-Key`.

Parameters:
- `id` (path) (required)
- `Idempotency-Key` (header) - A unique key you generate per logical create. Retrying with the same key returns the original result instead of creating a duplicate.


Request body:
  - `contact_id`: string (required)

Success: `201`

---

