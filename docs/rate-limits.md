# Rate limits

Per-key limits, the headers we return, and how to back off.

Each API key has its own rate limit. The default is **100 requests per minute**; some
keys are configured higher. When you exceed it you get `429` with a `Retry-After` header.

## Headers

Every response includes your current standing:

| Header | Meaning |
| --- | --- |
| `X-RateLimit-Limit` | Requests allowed per window |
| `X-RateLimit-Remaining` | Requests left in the current window |
| `X-RateLimit-Reset` | Unix seconds when the window resets |
| `Retry-After` | (on 429 only) seconds to wait before retrying |

## Backing off

On a `429`, wait `Retry-After` seconds, then retry with exponential backoff and jitter.
Do not hammer, repeated over-limit requests keep the window full.

```
if response.status == 429:
    sleep(response.headers["Retry-After"])
    retry()
```

## Staying under the limit

- Prefer one paginated list call over many single-record calls.
- Cache catalog and location data, it rarely changes.
- Use webhooks (coming soon) instead of polling for changes.
