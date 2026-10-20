# Booking Pro API - Endpoint Catalog

Base URL: `https://api.bookingpro.ai/api/v1`
Auth: `Authorization: Bearer bp_sk_live_...` on every request.
All money is in integer cents of the object's `currency` (uppercase ISO; each location has its own, see `GET /locations`). All times are ISO 8601 (UTC).

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
| Members | `checkin:members (resolve/check-in), read:memberships (visit history)` |
| Classes | `read:classes (read), write:classes (book/cancel/check-in)` |
| Webhooks | `read:webhooks (read), write:webhooks (create/delete/rotate secret)` |

---

## GET /ping

Scope: `(any valid key)`

**Verify a key.** Returns the identity a key resolves to. Use it to confirm a key works and to see which company, scopes, environment, and locations it is bound to.

Success: `200`

---

## GET /availability

Scope: `read:availability`

**List open slots.** Returns bookable start times for a service at a location on a given date (or date range). Honors the location's business hours, staff schedules, existing bookings, classes, rooms, and booking rules, exactly like the public booking page. Each slot is sized for the staff member who would do it (their own duration, waiting period and cleanup time for the service).

Without `technician_id`: at a location where every appointment needs a staff member, this is "any available": a time is listed when ANY staff member who performs the service is free, and the slot's `technician_id` is the staff member `POST /bookings` assigns when you book that start without naming one (the first free one in the location's booking priority order). At most 7 days per request in this mode. At a location that books appointments onto rooms, the slots come from the rooms and `technician_id` is null.

Every `start` returned here is accepted by `POST /bookings` (unless someone books it first, which returns `409`).

When the requested days have no open time, `next_available` is the first open slot after them (searched up to 60 days ahead, 28 in any-available mode), so you can offer the earliest time without a second call. It is also set to the first slot when there are slots. Pass `limit` to return only the earliest slots (for example `limit=20` for a text conversation).

Parameters:
- `location_id` (query) (required)
- `service_id` (query) (required)
- `date` (query) (required) - Start date (YYYY-MM-DD), interpreted in the location's timezone.
- `days` (query) - Number of days from `date` to include (1-31). Defaults to 1.
- `technician_id` (query) - Restrict to a specific staff member. Must be someone who performs the service at the location (otherwise `404 technician_not_found`).
- `limit` (query) - Return at most this many slots, earliest first (1-500). Omit for every slot in the range.

Success: `200`

---

## GET /services

Scope: `read:catalog`

**List services.** The bookable services offered, scoped to the key's locations: every active service assigned to each location and open for online booking, with that location's price and duration. Each location lists its own rows (the same service at two locations is two rows with two ids). The list is complete in one response (not paginated).

Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.

Success: `200`

---

## GET /services/{id}

Scope: `read:catalog`

**Retrieve a service.** The same object `GET /services` returns. A service that list hides (deleted, inactive, not offered at its location, or closed to online booking) is a `404`.

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

**List contacts.** Contacts are shared by every location of the company.

Parameters:
- `search` (query) - Fuzzy match on name, phone, or email. A name also matches when written with or without accents ("jose" finds "José"); those contacts are listed after the exact-spelling matches.
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

**List bookings.** Lists bookings, newest first by default. To find a client's appointments from what you know about them, pass `phone` and/or `email`: the client is matched the same way booking matches a contact (any stored phone format), and the response adds `contact_found` and `matched_contacts` (more than one person can share a phone). Each matched contact carries `client_status`: `returning` when they have a past visit (an appointment completed, checked in or passed), otherwise `new`. For a "what are my upcoming appointments" lookup use `from` = now, `order=asc` and `exclude_cancelled=true`.

Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.
- `phone` (query) - Only bookings of the contact(s) with this phone number (any format; at least 7 digits).
- `email` (query) - Only bookings of the contact(s) with this email (case-insensitive exact match).
- `exclude_cancelled` (query) - Leave out cancelled bookings.
- `order` (query) - Sort by start time, `desc` (newest first, the default) or `asc` (soonest first).
- `from` (query) - Include bookings starting on/after this date-time.
- `to` (query) - Include bookings starting before this date-time.
- `contact_id` (query)
- `limit` (query)
- `cursor` (query) - The `next_cursor` from a previous page.

Success: `200`

---

## POST /bookings

Scope: `read:bookings (read), write:bookings (create/cancel/reschedule)`

**Create a booking.** Books an appointment. The start is re-checked exactly the way the public booking page re-checks it: the staff member's working hours, the location's hours, existing bookings, classes, rooms, and the location's booking rules (same day, how far ahead, minimum notice). The visit is sized for the staff member doing it (their own duration, waiting period and cleanup time for the service). A taken or closed time returns `409 slot_unavailable`; a broken booking rule returns `422 booking_rule_violation`.

Without `technician_id`, at a location where every appointment needs a staff member, the first free staff member who performs the service (in the location's booking priority order) is assigned: the same person `GET /availability` named for that slot. At a location that books onto rooms, the appointment is booked onto a room with no staff member (the service must have a room there, else `422 service_not_bookable`).

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

**Cancel a booking.** Cancels the appointment the same way the dashboard does: it gets the business's Cancelled status, the customer and staff are notified, the calendar event is removed, and automations and webhooks (`booking.cancelled`) fire. Cancelling a booking that is already cancelled changes nothing and returns it.
Money: anything the customer already paid toward the appointment (a deposit) becomes their store credit with the business. When the business's late-cancellation policy applies, the fee is taken from that deposit first (a $100 deposit with a $25 late fee leaves $75 of credit). Nothing is refunded to a card automatically; the business can refund the credit or waive the fee from the dashboard.

Parameters:
- `id` (path) (required)

Request body:
  - `reason`: string - Optional cancellation reason (stored on the booking).

Success: `200`

---

## POST /bookings/{id}/reschedule

Scope: `read:bookings (read), write:bookings (create/cancel/reschedule)`

**Reschedule a booking.** Moves a booking to a new start time (and optionally to another staff member). The new start is re-checked exactly like create: working and location hours, existing bookings, classes, rooms, and the booking rules. When `technician_id` names a different staff member, the visit is re-sized for that person (their own duration, waiting period and cleanup time for each service); each service keeps its quantity, price and room. A cancelled booking cannot be rescheduled (`409 booking_cancelled`); create a new one.

Parameters:
- `id` (path) (required)

Request body:
  - `start`: string (required) - The new start. Use a start from `GET /availability`.
  - `technician_id`: string - Move the booking to this staff member (must perform its services at the location). Omit to keep the current one.

Success: `200`

---

## GET /gift-cards

Scope: `read:giftcards`

**List gift cards.** Gift cards issued at the key's locations (a location-limited key does not see cards of other locations, or cards that belong to no single location). Pass `location_id` to narrow to one of the key's locations.

Parameters:
- `location_id` (query) - Restrict to a single location. Must be one the key can access.
- `limit` (query)
- `cursor` (query) - The `next_cursor` from a previous page.

Success: `200`

---

## GET /gift-cards/{id}

Scope: `read:giftcards`

**Retrieve a gift card.** A gift card outside the key's locations answers `404`, exactly like one that does not exist (same rule as the list).

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

**List a contact's memberships.** The memberships a contact holds (as primary member or family member), with status and remaining credits. A location-limited key sees only memberships whose home location is one of its locations.

Parameters:
- `id` (path) (required)

Success: `200`

---

## POST /members/resolve

Scope: `checkin:members (resolve/check-in), read:memberships (visit history)`

**Resolve a member and check eligibility.** Given a credential (a member code or an RFID fob/card UID) and a location, returns the matching membership and whether the member is currently eligible to check in. Use this from a kiosk, turnstile, or a biometric device (map a face/fingerprint to the member's code, then call this) to decide whether to let someone in, without recording a visit. The answer runs every rule `POST /members/check-in` runs (membership active, facility plan, blackout window, location covered, home-location-only, the location's check-in blocks, credits, and the currency rule), so `valid_for_checkin: true` means the check-in will be accepted.

Request body:
  - `identifier`: string (required) - The member's code or fob/card UID.
  - `location_id`: string (required)

Success: `200`

---

## POST /members/check-in

Scope: `checkin:members (resolve/check-in), read:memberships (visit history)`

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

Scope: `checkin:members (resolve/check-in), read:memberships (visit history)`

**List a membership's check-ins.** A location-limited key sees only memberships whose home location is one of its locations (another answers `404`), and only the check-ins made at its locations.

Parameters:
- `id` (path) (required)
- `limit` (query)

Success: `200`

---

## GET /classes

Scope: `read:classes (read), write:classes (book/cancel/check-in)`

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

Scope: `read:classes (read), write:classes (book/cancel/check-in)`

**Retrieve a class with its roster.** 
Parameters:
- `id` (path) (required)

Success: `200`

---

## POST /classes/{id}/book

Scope: `read:classes (read), write:classes (book/cancel/check-in)`

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

Scope: `read:classes (read), write:classes (book/cancel/check-in)`

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

Scope: `read:classes (read), write:classes (book/cancel/check-in)`

**Check a booked contact in.** Marks the contact's spot checked in and consumes what covers it (a membership visit or a package credit). Honors the location's member check-in alert blocks (422 `checkin_blocked` with `reasons`; a manager overrides at the desk). Send an `Idempotency-Key`.

Parameters:
- `id` (path) (required)
- `Idempotency-Key` (header) - A unique key you generate per logical create. Retrying with the same key returns the original result instead of creating a duplicate.


Request body:
  - `contact_id`: string (required)

Success: `201`

---

## GET /webhooks

Scope: `read:webhooks (read), write:webhooks (create/delete/rotate secret)`

**List webhook endpoints.** Returns the webhook endpoints registered for your account. The signing secret is never returned here; it is shown only once, when the endpoint is created (or its secret rotated). A location-limited key sees and manages only endpoints limited to its own locations (never an all-locations endpoint); any other endpoint answers `404` to it on every webhook route.

Success: `200`

---

## POST /webhooks

Scope: `read:webhooks (read), write:webhooks (create/delete/rotate secret)`

**Register a webhook endpoint.** Register an https URL to receive events. Booking Pro POSTs a JSON body for each subscribed event, signed with HMAC-SHA256 in the `X-BookingPro-Signature` header (`sha256=<hex>`) using the `secret` returned here. Verify every delivery by recomputing the HMAC of the raw request body with that secret. Each delivery also carries a unique `X-BookingPro-Event-Id` you can use to deduplicate. The `secret` is shown ONLY in this response. Subscribe to specific event types, or `"*"` for all. Send an `Idempotency-Key` so a retried create does not register twice. The body of every delivery is a `WebhookEvent`; payment events carry `amount_cents`, `tip_cents` and their `currency` (uppercase ISO 4217).

Parameters:
- `Idempotency-Key` (header) - A unique key you generate per logical create. Retrying with the same key returns the original result instead of creating a duplicate.


Request body:
  - `url`: string (required) - An https:// URL that will receive POSTed events.
  - `event_types`: array (required) - Events to receive, or ["*"] for all.
  - `description`: string

Success: `201`

---

## GET /webhooks/{id}

Scope: `read:webhooks (read), write:webhooks (create/delete/rotate secret)`

**Get a webhook endpoint.** 
Parameters:
- `id` (path) (required)

Success: `200`

---

## DELETE /webhooks/{id}

Scope: `read:webhooks (read), write:webhooks (create/delete/rotate secret)`

**Delete a webhook endpoint.** Permanently removes the endpoint. Pending deliveries to it are dropped.

Parameters:
- `id` (path) (required)

Success: `200`

---

## POST /webhooks/{id}/rotate-secret

Scope: `read:webhooks (read), write:webhooks (create/delete/rotate secret)`

**Rotate a webhook's signing secret.** Generates a new signing secret and returns it ONCE. Update your receiver to verify with the new secret. The old secret stops working immediately.

Parameters:
- `id` (path) (required)

Success: `200`

---

## GET /webhooks/{id}/deliveries

Scope: `read:webhooks (read), write:webhooks (create/delete/rotate secret)`

**List recent deliveries for a webhook.** The delivery log for an endpoint: one row per event sent to it, with the delivery status, attempt count, last HTTP response status, and any error. Use it to debug a receiver that is not acknowledging events.

Parameters:
- `id` (path) (required)
- `limit` (query)

Success: `200`

---

