---
name: ginnie-mae-answer-from-official-sources
description: >-
  Answer a question about Ginnie Mae — a program rule, a term of art, a policy change —
  from Ginnie Mae's own published text rather than from memory, using the agency's
  public JSON:API content surface. Use when someone asks what Ginnie Mae's position,
  definition or requirement actually is and the answer must be citable.
api: ginnie-mae:ginnie-mae-content-api
generated: '2026-09-12'
method: generated
source: openapi/ginnie-mae-content-api-openapi.yml
operations:
  - searchSite
  - listFaqs
  - listFaqCategories
  - listFaqSubcategories
  - listGlossaryTerms
  - listNodeMemorandum
  - getNodeMemorandum
  - listMediaDocumentFile
  - getFileFile
---

# Answer a Ginnie Mae question from official sources

Base URL: `https://www.ginniemae.gov/api/v1`. No authentication. No API key. Send nothing
but the request.

## When to use this

Someone asks what Ginnie Mae requires, defines, or announced. Ginnie Mae publishes no
documentation for this API, but the text on ginniemae.gov is reachable through it, so the
answer can be sourced rather than recalled.

## Steps

1. **Try the glossary first for a term of art.** `listGlossaryTerms` — `GET /api/v1/glossary`
   returns all 174 terms as a bare array of `{title, field_term_definition}`. It is small
   enough to fetch whole and match locally. If the question is "what is an HMBS pool" or
   "what does RPB mean", the answer is here and it is Ginnie Mae's own wording.

2. **Try the FAQ for a program question.** `listFaqs` — `GET /api/v1/faq?keywords=<terms>`.
   The response is `{filters, pager, results}` where each result carries
   `question`, `answer`, `modified`, `reference` and `category`. Read `pager.total_results`
   before paging. Narrow with `category=` and `subcategory=`, whose valid values come from
   `listFaqCategories` (`GET /api/v1/faq/categories`) and `listFaqSubcategories`
   (`GET /api/v1/faq/subcategories`).

3. **Fall back to site-wide search.** `searchSite` — `GET /api/v1/search/results?keywords=<terms>`.
   Results carry `title`, `url`, `file_extensions` and `search_api_excerpt` (the matched
   passage, already highlighted). Use `result_type=` to narrow and `sort_by`/`sort_order`
   to order. The endpoint declares its own accepted filters in the response `filters`
   member — read that rather than guessing.

4. **For a policy change, go to the memoranda.** `listNodeMemorandum` —
   `GET /api/v1/node/memorandum?sort=-created&page[limit]=25`. All Participant Memoranda
   (APMs), MBS Program Memoranda (MPMs) and bulletins are the authoritative change record.
   Filter with `filter[title][operator]=CONTAINS&filter[title][value]=<phrase>`.

5. **Retrieve the underlying PDF when the answer needs it.** A memorandum references its
   attachment through `field_featured_media` as an inline `{type: "media--document_file", id: "<uuid>"}`
   object. Resolve it with `getMediaDocumentFile`, then follow that media record's
   `field_media_file` to a `file--file` record via `getFileFile`, which carries the download
   URI. Or do it in one request with `include=field_featured_media.field_media_file` and read
   the `included` array.

6. **Cite the human page, not the API path.** Every content record carries a `path` member
   with the site alias. Give the reader `https://www.ginniemae.gov<path>`.

## Rules that will bite you

- **Attributes are flattened.** This site runs JSON:API Extras with field enhancement.
  There is NO `data.attributes` member — fields sit directly on the resource object. A
  vanilla JSON:API client reads every record as empty. Fetch one record with
  `page[limit]=1` and read the real key names before writing a filter.
- **Filter on a field that does not exist and you get HTTP 400**, with a JSON:API
  `errors[]` document whose `detail` names the offending field. That message is accurate —
  use it.
- **An unknown resource type returns an HTML 404 page, not JSON.** Guard the parse.
- **No rate limit is published and no rate-limit header is returned.** You get no warning
  before you are cut off. Keep concurrency low, prefer one `include=` request over three
  round trips, and cache the glossary and FAQ category lists — they change rarely.
- **Do not enumerate `user--user`.** It is CMS account data, not published content.
- `robots.txt` on this host carries `Disallow: /api/`. The surface is open and
  unauthenticated, but it is not an advertised product — behave accordingly.
