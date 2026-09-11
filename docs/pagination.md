# Pagination

How to page through list endpoints with cursors.

List endpoints return a page of results plus a cursor for the next page:

```json
{
  "data": [ { "id": "..." }, { "id": "..." } ],
  "next_cursor": "eyJvIjoyNX0"
}
```

- Pass `?limit=` to set the page size (1-100, default 25).
- Pass `?cursor=` with the previous response's `next_cursor` to get the next page.
- When `next_cursor` is `null`, you have reached the end.

```bash
# first page
curl "https://api.bookingpro.ai/api/v1/contacts?limit=50" \
  -H "Authorization: Bearer bp_sk_live_..."

# next page
curl "https://api.bookingpro.ai/api/v1/contacts?limit=50&cursor=eyJvIjoyNX0" \
  -H "Authorization: Bearer bp_sk_live_..."
```

Cursors are opaque, do not construct or parse them. Loop until `next_cursor` is `null`
to drain a full collection.
