---
name: twinstrand-biosciences-patent-estate
description: >-
  Pull the Duplex Sequencing patent estate TwinStrand Biosciences publishes as JSON — 28 patent
  records addressable by patent number — plus the team roster, over the public WordPress REST API.
generated: '2026-08-05'
method: generated
source: openapi/twinstrand-biosciences-content-openapi.yml
api: twinstrand-biosciences:content
base_url: https://twinstrandbio.com/wp-json/wp/v2
auth: none (anonymous read)
operations:
  - listPatent
  - getPatent
  - listTeamMember
  - getTeamMember
  - listPages
  - getPages
---

# Patent estate and company roster

## The patent collection

TwinStrand exposes its patent estate as a first-class JSON collection — unusual, and useful given
the 2024 federal verdict on the University of Washington Duplex Sequencing patents it exclusively
licenses.

```
GET /wp/v2/patent?per_page=100&_fields=id,date,slug,link,title,content
```

`X-WP-Total` was 28 on 2026-08-05. The **`title.rendered` and `slug` are the raw patent number**
(e.g. `3601598`, `6975507`), so you can address a record directly:

```
GET /wp/v2/patent?slug=6975507
```

Detail (claims, jurisdiction, dates) lives in `content.rendered` as HTML — there is no structured
patent schema. Cross-reference the human page at https://twinstrandbio.com/legal/patents/.

## Team roster

```
GET /wp/v2/team-member?per_page=100&_embed&_fields=id,slug,link,title,content,categories,featured_media
```

10 profiles. `categories` are term ids in the shared `category` taxonomy (4 terms — resolve them
with `GET /wp/v2/categories`), and `_embed` inlines the headshot from the media library. Only one
author is exposed on `/wp/v2/users`, so do not expect per-profile authorship.

## Company pages

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,parent
```

24 pages including `/technology/`, `/aml-assay/`, `/mutagenesis-assay/`, `/nitrosamine-testing/`
and the `/legal/*` set. `parent` is the page-hierarchy pointer (0 at the top level), so you can
rebuild the site tree from one call.

## Rules

- **Content-Type first.** An unknown id returns a `text/html` 404 page from the edge, not JSON.
- `content.rendered` is HTML written for humans. Treat it as text to extract from, not as a schema.
- Nothing about the assay data, sequencing runs or the DuplexSeq analysis pipeline is reachable
  here. That software runs on the DNAnexus platform and is customer-only — there is no public
  endpoint for it, and no amount of walking this API will find one.
- Content is © TwinStrand Biosciences and the patents are its property; this API grants no license.
