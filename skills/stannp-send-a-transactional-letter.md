---
name: Send a transactional Stannp letter
description: >-
  Post a single letter through the Stannp Direct Mail API — either mail-merged
  from a template or HTML, or dispatched as a pre-merged PDF whose address is
  read by OCR — with test-mode proofing and idempotent retries.
api: openapi/stannp-letters-api-openapi.yml
operations:
  - createLetter
  - postLetter
  - getLetter
  - cancelLetter
generated: '2026-08-13'
method: generated
source: https://www.stannp.com/us/direct-mail-api/letters
---

# Send a transactional Stannp letter

Two operations post a letter, and choosing the wrong one is the most common
mistake on this API.

| You have | Use | Address comes from |
|---|---|---|
| A template or HTML plus recipient data | `createLetter` (`POST /v1/letters/create`) | the `recipient` you supply, mail-merged |
| A finished PDF that already has the address printed on it | `postLetter` (`POST /v1/letters/post`) | OCR of the document |

## Setup

- Host: `https://api-us1.stannp.com/v1` (US/CA) or
  `https://api-eu1.stannp.com/v1` (EU/UK).
- Auth: API key as the HTTP Basic username, empty password.
- Body: form-encoded (`application/x-www-form-urlencoded` or
  `multipart/form-data`). Never JSON.

## Path A — mail merge with `createLetter`

Supply `recipient` (an existing recipient ID, or inline
`recipient[address1]`, `recipient[city]`, `recipient[postcode]`,
`recipient[country]`, plus any custom `recipient[*]` fields you reference as
merge variables) and one of:

- `template=<template id>` — a design stored on the account, or
- `pages=<HTML>` with an optional `background=<URL>`, or
- `file=<PDF/DOC URL or upload>` — maximum 10 pages.

`size` is `US-LETTER` on a US account and `A4` on an EU/UK account; both default
correctly for the account region. `duplex` defaults to true. `addons` accepts
`FIRST_CLASS`, `C4_ENVELOPE`, `CONFIDENTIAL`, `STOCK_120`, comma-separated.

Run it once with **`test=true`** first. You get back
`{"success": true, "data": {"pdf": "...", "id": 0, "status": "test", ...}}` —
open the PDF, check the merge and the window position, then repeat without
`test` and with `idempotency_key=<persisted UUID>`.

## Path B — pre-merged PDF with `postLetter`

`country` is **mandatory** here (ISO 3166-1 alpha-2: `US`, `CA`, `GB`, ...), and
`pdf` is a URL or binary upload of a document up to 25 pages. Stannp OCRs the
address off the page, so the address block must sit correctly in the window
zone; prove it with `test=true` before going live.

Set `transactional=true` when the letter carries sensitive data — it routes the
piece through Stannp's handling path for confidential material and is reflected
back on the mailpiece object as `transactional`.

## Responses

- **200** — created. Persist `data.id`.
- **409** — an idempotent replay. **Treat it as success**: the body is the
  original response, the letter already exists, and retrying again risks a
  duplicate physical letter if you drop the key.
- **401** — bad key, wrong regional host, or plain HTTP.
- **404** — on `getLetter`, the mailpiece ID does not exist.
- **500** — retry with backoff, reusing the same `idempotency_key`.

## Track and cancel

`getLetter` (`GET /v1/letters/get/{id}`) returns the mailpiece object with
`status`, `dispatched`, `tracking_ref` and a `tracking` array. `cancelLetter`
(`POST /v1/letters/cancel`, `id=<mailpiece id>`) only succeeds before production
starts.

For status changes, subscribe to the `mailpiece_status` webhook rather than
polling; verify the `X-Stannp-Signature` HMAC-SHA256 over the raw body with a
constant-time comparison. See `asyncapi/stannp-webhooks.yml`.

## Compliance note

Stannp is HIPAA-compliant as a business associate and ICO-registered as a UK
data processor (`conformance/stannp-conformance.yml`). If the letter contains
PHI or other regulated content, set `transactional=true` and confirm your own
BAA is in place — the API will happily post it either way.
