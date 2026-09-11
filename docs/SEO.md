# SEO and AI-discoverability decisions

This file records the search and AI-discoverability work on the Booking Pro API
docs, with the reasoning behind each change. Read it before changing
`robots.txt`, `sitemap.xml`, canonical logic, structured data, or public copy.

The docs site is built from the `api-docs/` app in the main monorepo and deploys
to `developers.bookingpro.ai` (Cloudflare Pages, from `dev`). This public repo
mirrors the spec, the guides, and the agent skill.

## 2026-09-11: Made the docs site and repo findable by Google and AI models

**The situation.** The docs site is a client-rendered SPA (Fumadocs on React
Router, `ssr:false`) that prerenders every docs page to static HTML. It went
live with no `robots.txt` and no `sitemap.xml`: requests for those paths fell
through to the SPA fallback and returned the app's HTML shell. A `robots.txt`
that is really HTML reads as "no rules", and there was no sitemap pointer at all.

**What I verified first (the good news):** the prerendered HTML is identical for
a non-JS crawler and a browser (tested ClaudeBot, OAI-SearchBot, PerplexityBot,
Googlebot vs a browser UA: same ~63 KB, same text blocks). So the content is
genuinely readable by AI-retrieval crawlers that do not run JavaScript, which is
the failure that usually kills AI discoverability on an SPA. Titles and
descriptions are present and unique. `llms.txt` / `llms-full.txt` are live.

**What I changed (all in `api-docs/` in the monorepo):**

- `app/seo/robots.ts` + route in `app/routes.ts`: serves a real `/robots.txt`
  (prerendered static file) allowing every crawler (`User-agent: *`, `Allow: /`)
  and pointing to the sitemap. This is a public docs site whose whole purpose is
  to be found and cited, so every search AND AI-retrieval bot is allowed; we do
  not use the per-bot training opt-out template, which is for sites declining
  training. A single `*` group avoids the precedence trap where named groups do
  not inherit the `*` rules.
- `app/seo/sitemap.ts` + route: serves a real `/sitemap.xml` generated from the
  same page source as the docs (`source.getPages()`), so it never drifts. Home,
  `/docs`, all guides, all generated API-reference pages. Absolute URLs only.
- `app/root.tsx`: `Organization` + `WebSite` JSON-LD in the document head, so
  search engines and AI systems can recognize the entity. Every fact in it is
  also visible on the site (rule: structured data must not assert what the page
  does not show).
- `app/lib/layout.shared.tsx` + `content/docs/index.mdx`: a "View on GitHub"
  link in the nav and on the landing page, pointing at this public repo.

**Why it never drifts:** robots and sitemap are resource routes prerendered at
build time from the same source of truth as the pages, not hand-maintained
files. Adding a docs page adds it to the sitemap automatically.

**Still needs a human (GitHub repo settings, cannot be done from code):**

- Set the repo **description**, **homepage** (`https://developers.bookingpro.ai`),
  and **topics** (e.g. `api`, `rest-api`, `openapi`, `booking`, `scheduling`,
  `salon-software`, `agent-skill`). These three fields are how the repo itself is
  found on GitHub and in search results; they are currently empty.

**To undo any of this:** delete the two `app/seo/*.ts` files and their lines in
`app/routes.ts` (reverts robots/sitemap to the SPA fallback), or remove the
`STRUCTURED_DATA` block in `app/root.tsx`.
