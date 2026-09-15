# Classes

Read the class timetable, see who is booked, and book, cancel or check in a spot.

Gyms and studios run **group classes** next to one-to-one appointments: a class
has spots, a roster, and a waitlist, and a member's spot is covered by their
membership or package when they check in. These endpoints need the
`read:classes` and `write:classes` scopes. A salon or spa key gets an empty
timetable.

## The timetable

`GET /classes` lists the classes in a window (default the next 7 days, at most
31), earliest first, with how many spots are booked and left:

```bash
curl "https://api.bookingpro.ai/api/v1/classes?location_id=loc_...&from=2026-09-16T00:00:00Z&to=2026-09-17T00:00:00Z" \
  -H "Authorization: Bearer bp_sk_live_..."
```

```json
{
  "data": [
    {
      "id": "cls_...",
      "location_id": "loc_...",
      "service_id": "svc_...",
      "service_name": "Spin Class",
      "technician_name": "Dana Levi",
      "start": "2026-09-16T17:00:00Z",
      "end": "2026-09-16T17:45:00Z",
      "capacity": 16,
      "booked": 11,
      "spots_left": 5,
      "waiting": 0,
      "status": "scheduled"
    }
  ],
  "next_cursor": null
}
```

`GET /classes/{id}` adds the roster (each spot with its `status` and what covers
it in `paid_by`) and the waitlist in order.

## Book a spot

`POST /classes/{id}/book` with a `contact_id` holds a spot. Nothing is charged at
booking; `paid_by` tells you what will be consumed at check-in:

- `membership`: a plan that covers the service with a visit available (per-service
  limits such as "2 classes a week" are honored by class start time).
- `package`: a prepaid credit.
- `free_class`: one of the free classes the service allows before payment.
- `none`: nothing covers it; the front desk charges at checkout.

When the service **requires** a membership or package and the contact has none,
you get `422 entitlement_required`. Pass `allow_without_entitlement: true` to
book anyway (the desk charges later), which is what staff do. A full class
returns `409 class_full`; members can join the waitlist from the customer app.
Signup cutoff, max per day and first-timers-only rules return `422 booking_rule`
with a plain-language message. Send an `Idempotency-Key`.

## Cancel a spot

`POST /classes/{id}/cancel` with the `contact_id` releases the spot. Inside the
location's cancellation window it is a **late cancel** (`late: true`) and the
covering credit is still consumed, unless you pass `excuse: true`. A freed spot
goes to the waitlist automatically.

## Check in

`POST /classes/{id}/check-in` with the `contact_id` marks the spot checked in and
consumes the covering credit. The location's member check-in alert blocks apply:
a blocked check-in returns `422 checkin_blocked` with `reasons` (for example
`past_due`, `no_card`, `form_missing`); a manager overrides at the desk.

## What you cannot do here yet

Creating class times (the weekly series), the waitlist, and standing enrollments
are managed in the dashboard and the customer app. Tell us if your integration
needs them.
