---
name: twinstrand-biosciences-publication-index
description: >-
  Walk the TwinStrand Biosciences publication index over the public WordPress REST API — list the
  38 peer-reviewed Duplex Sequencing papers, resolve the topical and chronological taxonomies, and
  filter by either. No credentials required.
generated: '2026-08-05'
method: generated
source: openapi/twinstrand-biosciences-content-openapi.yml
api: twinstrand-biosciences:content
base_url: https://twinstrandbio.com/wp-json/wp/v2
auth: none (anonymous read)
operations:
  - listPublication
  - getPublication
  - listPublicationCategory
  - getPublicationCategory
  - listPublicationYear
  - getPublicationYear
---

# TwinStrand publication index

TwinStrand Biosciences keeps a curated index of the peer-reviewed literature on Duplex Sequencing.
It is served as JSON from the site's WordPress REST API and needs no credentials.

## 1. Resolve the taxonomies first

Publications are joined to two taxonomies by integer term id, so resolve them before filtering.

```
GET /wp/v2/publication-category?per_page=100&_fields=id,name,slug,count
GET /wp/v2/publication-year?per_page=100&_fields=id,name,slug,count
```

`publication-category` has 11 terms (Genomic Toxicology, Mutagenesis, Oncology, AML MRD,
Carcinogenesis, Gene Therapy, Hematological Malignancy, TP53, Duplex Sequencing Technology, Custom,
Featured); `publication-year` has 14. Cache the id→name maps — nothing in the publication payload
carries the term names.

## 2. Page the collection

```
GET /wp/v2/publication?per_page=100&orderby=date&order=desc&_fields=id,date,slug,link,title,publication-category,publication-year
```

`per_page` is capped at 100 (a larger value returns `400 rest_invalid_param`, not a clamp), so at
38 items one page is enough today. Read `X-WP-Total` and `X-WP-TotalPages` from the response
headers rather than counting, and follow the RFC 8288 `Link: rel="next"` header if the collection
grows.

## 3. Filter

```
GET /wp/v2/publication?publication-category=19&per_page=100   # Genomic Toxicology
GET /wp/v2/publication?publication-year=66&per_page=100
GET /wp/v2/publication?search=mutagenesis&per_page=100
```

`publication-category_exclude` and `publication-year_exclude` are the negations, and
`tax_relation=AND|OR` controls how the two taxonomies combine.

## 4. Fetch one, with the featured asset inlined

```
GET /wp/v2/publication/{id}?_embed
```

`_embed` inlines `wp:featuredmedia` and `wp:term` into `_embedded` in one round trip instead of
walking `featured_media` and the term ids yourself.

## Rules

- **Branch on Content-Type before parsing.** An unknown id returns a `text/html` 404 page from the
  edge, not the WordPress JSON error envelope. `GET /wp/v2/publication/999999` is the proof.
- Errors that *are* JSON use `{code, message, data.status}` — not RFC 9457 problem+json.
- There is no idempotency-key contract and no rate-limit header. Every operation here is a GET;
  be a polite client anyway — nothing on this host publishes a quota.
- `acf` is present on every resource and was empty on the sampled publication. Do not depend on it.
