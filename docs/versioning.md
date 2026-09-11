# Versioning

How we version the API and what counts as a breaking change.

The API is versioned in the URL path:

```
https://api.bookingpro.ai/api/v1
```

## Our promise

Within `v1` we only make **additive** changes:

- New endpoints.
- New optional request fields.
- New fields in responses.

Your integration should ignore unknown response fields so these additions never break it.

## Breaking changes

A breaking change (removing a field, renaming one, changing a type, or changing required
inputs) ships under a new version, `v2`, and `v1` keeps working. We announce new versions
and deprecations well in advance in the [changelog](./changelog.md).
