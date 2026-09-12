---
name: ginnie-mae-navigate-disclosure-data
description: >-
  Find out which Ginnie Mae MBS disclosure file answers a question, and what its record
  layout is, before trying to download anything. Use when someone needs pool-level,
  loan-level, HMBS, Multifamily, REMIC or Platinum disclosure data and needs to know
  which file to ask for and how it is structured.
api: ginnie-mae:ginnie-mae-content-api
generated: '2026-09-12'
method: generated
source: openapi/ginnie-mae-content-api-openapi.yml
operations:
  - listNodeDataFileTypes
  - getNodeDataFileTypes
  - listMediaDisclosureDataFile
  - getMediaDisclosureDataFile
  - getFileFile
  - searchSite
  - listBulkNotifications
  - getMenuLinkset
---

# Navigate Ginnie Mae disclosure data

Base URL: `https://www.ginniemae.gov/api/v1`. No authentication.

## Read this before you start

**The disclosure data itself is not in this API.** Ginnie Mae distributes security- and
loan-level MBS disclosure as bulk files through the Disclosure Data Download application
at `https://bulk.ginniemae.gov/`, and downloading a file requires a free registered
account (sign up at `https://www.ginniemae.gov/disclosure/create-account`). What the
public API gives you is the **catalogue and the record layouts** — which is the part an
agent actually needs first, because a Ginnie Mae disclosure file is fixed-width and
unreadable without its layout.

Do not attempt to fetch files from `https://www.ginniemae.gov/disclosure-api/api/...`
anonymously. It answers the Angular application shell with HTTP 200, which is not data
and is not an error either.

## Steps

1. **List the file types.** `listNodeDataFileTypes` —
   `GET /api/v1/node/data_file_types?page[limit]=50`. Each record describes one disclosure
   data file type. Fetch one record first and read its real field names; attributes are
   flattened onto the resource object, not nested under `attributes`.

2. **Resolve the record layout.** Each file type references its published layout document
   through `field_file_layout`, an inline `{type: "media--disclosure_data_file", id: "<uuid>"}`
   object. Ask for it in the same request:
   `GET /api/v1/node/data_file_types?include=field_file_layout.field_media_file` and read
   the `included` array, or resolve it in two hops with `getMediaDisclosureDataFile` then
   `getFileFile`. Note that `media--disclosure_data_file` also carries a
   `field_secondary_file`, so a layout may come as a pair of documents.

3. **Cross-check against the human page.** The published index is
   `https://www.ginniemae.gov/disclosure/disclosure-data/disclosure-data-download-files`.
   Use `getMenuLinkset` with `menu=disclosure-plus-download-data` to get that whole section
   as an RFC 9264 linkset if you need the full shape of it.

4. **Check for operational notices.** `listBulkNotifications` —
   `GET /api/v1/bulk-notifications` returns the notice banner the download application
   shows: republished files, corrections, outages. It returned an empty array on
   2026-09-12, which means no active notice, not a missing endpoint.

5. **Search for corrections and republications.** Data corrections are announced as
   bulletins. `searchSite` with `keywords=corrected` plus the pool or file name, or
   `GET /api/v1/node/memorandum?filter[title][operator]=CONTAINS&filter[title][value]=corrected&sort=-created`.
   This matters: Ginnie Mae republishes files, and a stale download is a real failure mode.

6. **Then hand off to a human for the download.** The account, the login and the file
   retrieval are outside what an agent can do unattended here. Report the file type, the
   layout document URL and the download application URL, and stop.

## Rules that will bite you

- **Nothing here is a security price, a factor or a balance.** This API returns the
  catalogue and the documents. Never synthesise a disclosure value from it.
- **Attributes are flattened** — no `data.attributes`. Read a live record first.
- **An unknown resource type returns HTML, not a JSON:API error.** Guard the parse.
- **No rate limit is published and no rate-limit header is returned.** Keep concurrency low.
