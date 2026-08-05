---
name: twinstrand-biosciences-newsroom-and-events
description: >-
  Track TwinStrand Biosciences announcements and appearances over the public WordPress REST API —
  27 news releases, 30 events and 14 conference posters — including how to poll for new items
  without a feed, since the site's RSS is empty and the per-post-type feed is 403.
generated: '2026-08-05'
method: generated
source: openapi/twinstrand-biosciences-content-openapi.yml
api: twinstrand-biosciences:content
base_url: https://twinstrandbio.com/wp-json/wp/v2
auth: none (anonymous read)
operations:
  - listNews
  - getNews
  - listEvent
  - getEvent
  - listPoster
  - getPoster
---

# TwinStrand newsroom, events and posters

## Why the API and not a feed

`https://twinstrandbio.com/feed/` returns 200 but zero items — the `post` collection is empty and
there is no blog. The per-post-type feed at `/company/news/feed/` returns **403**. The REST
collections are the only working way to watch this company programmatically.

## Latest announcements

```
GET /wp/v2/news?per_page=10&orderby=date&order=desc&_fields=id,date,modified,slug,link,title,excerpt
```

`X-WP-Total` was 27 on 2026-08-05. The most recent entry was the 2026-02-26 OECD regulatory-approval
release.

## Poll for new items without refetching everything

```
GET /wp/v2/news?after=2026-02-26T00:00:00&orderby=date&order=asc&_fields=id,date,link,title
GET /wp/v2/news?modified_after=<last-run-iso8601>&_fields=id,modified,link,title
```

`after` / `before` filter on publish date; `modified_after` / `modified_before` catch edits to
already-published items. Store the high-water mark from `modified` (site timezone is
America/Denver, `gmt_offset` -6) and use `modified_after` on subsequent runs.

## Events and posters

```
GET /wp/v2/event?per_page=100&orderby=date&order=desc&_fields=id,date,slug,link,title
GET /wp/v2/poster?per_page=100&_embed&_fields=id,date,slug,link,title,featured_media
```

Posters usually carry a PDF as `featured_media`; `_embed` inlines the attachment so you get
`_embedded["wp:featuredmedia"][0].source_url` without a second call. Media lives at
`/wp/v2/media/{id}` if you would rather resolve it explicitly (370 attachments total).

## One search across all of it

```
GET /wp/v2/search?search=duplex&per_page=20
```

Branch on `subtype` (`news`, `poster`, `publication`, `patent`, `event`, `team-member`, `page`) —
`type` is always `post` and tells you nothing. `_links.self[0].href` on each hit is the direct
collection URL to follow.

## Rules

- **Content-Type first.** Unknown ids and routes return a `text/html` 404 from the edge, not JSON.
- `per_page` max is 100; 101 or more is a `400 rest_invalid_param`, not a clamp.
- No rate-limit headers are published. Poll on a human cadence — daily is more than enough for a
  newsroom that posted three times in two years.
- Content is © TwinStrand Biosciences; the API carries no license grant. See
  https://twinstrandbio.com/legal/.
