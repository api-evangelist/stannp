---
name: Send a single Stannp postcard
description: >-
  Create and dispatch one physical postcard to an address through the Stannp
  Direct Mail API, proving the artwork in test mode first and using an
  idempotency key so a retry cannot produce a second piece of mail.
api: openapi/stannp-postcards-api-openapi.yml
operations:
  - createPostcard
  - getPostcard
  - cancelPostcard
  - getAccountBalance
generated: '2026-08-13'
method: generated
source: https://www.stannp.com/us/direct-mail-api/postcards
---

# Send a single Stannp postcard

Stannp prints and posts real mail. Every step below has a physical, irreversible
consequence once a mailpiece enters production, so the order matters.

## Before you start

- **Host by region.** Use `https://api-us1.stannp.com/v1` for a US/CA account and
  `https://api-eu1.stannp.com/v1` for an EU/UK account. A key issued in one
  region returns 401 against the other host — that 401 is not a bad key.
- **Auth.** Send the API key as the HTTP Basic username with an empty password
  (`-u {API_KEY}:`). The `api_key` query-parameter form also works but leaks the
  key into logs; prefer Basic.
- **Encoding.** This is a form-encoded API. Send
  `application/x-www-form-urlencoded` or `multipart/form-data`, never a JSON
  body. Nested values use bracket notation: `recipient[firstname]=John`.

## 1. Confirm there is money to spend

Call `getAccountBalance` (`GET /v1/accounts/balance`). It returns
`{"success": true, "data": {"balance": "100.00"}}`. A live send draws from this
balance; if it is short, the send will not go out.

## 2. Prove the artwork in test mode

Call `createPostcard` (`POST /v1/postcards/create`) with **`test=true`**.

Supply either a `template` ID, or `front` and `back` image URLs. Supply
`recipient` as either an existing recipient ID or an inline
`recipient[address1]`, `recipient[city]`, `recipient[postcode]`,
`recipient[country]` set. `size` is `4x6`, `6x9` or `6x11` on a US account and
`A6`, `A5`, `A5-ENV` or `A5-PORT` on an EU/UK account.

The response carries `"status": "test"`, `"id": 0` and a `pdf` URL. Open that PDF
and check the mail merge, the clear zones and the address block. Nothing was
dispatched and nothing was charged.

**`test` defaults to false.** Omitting it is the difference between a proof and a
posted postcard.

## 3. Send it for real, with an idempotency key

Repeat the same call **without `test`** (or with `test=false`) and add
`idempotency_key=<a UUID you generate and persist>`.

Stannp compares the *full request body* against the key, so the retry body must
be byte-identical. Persist the key next to your own record of the send before you
issue the request — a key you cannot recover after a timeout is a key that cannot
protect you.

Useful additions on the live call:

- `post_unverified=false` — refuse to post if the address could not be verified.
- `tags=order-1234,welcome` — free-text tags, later searchable in reporting.
- `addons=FIRST_CLASS` — upgrade codes, comma-separated.

## 4. Handle the response correctly

- **200** — the mailpiece was created. Store `data.id`; it is the only handle you
  get. Live responses also carry `tracking_ref`, `production_ref`, `cost`,
  `dispatched`, `status` and `class`, none of which appear in a test response.
- **409** — **this is success, not failure.** A duplicate idempotent request was
  detected and the body of this response is the *original* result. Do not retry
  again, and do not fall back to a non-idempotent create; that is how duplicate
  mail gets posted.
- **401** — bad key, wrong region host, or a plain-HTTP request.
- **500** — retry with backoff, sending the *same* `idempotency_key`.

There is no machine-readable error code. The body is
`{"success": false, "error": "<free text>"}`; branch on the HTTP status and treat
`error` as opaque.

## 5. Track it

Call `getPostcard` (`GET /v1/postcards/get/{id}`) for the mailpiece object. The
status ladder is `received` → `producing` → `handed_over` → `local_delivery` →
`delivered`, plus `returned` and `cancelled`.

Prefer webhooks over polling: subscribe to `mailpiece_status` and verify the
`X-Stannp-Signature` HMAC-SHA256 digest of the raw body. See
`asyncapi/stannp-webhooks.yml`. Webhooks are a paid entitlement — 0 on Free and
Starter, 2 on Growth, 4 on Premium.

## 6. Cancel only if you are quick

`cancelPostcard` (`POST /v1/postcards/cancel`, `id=<mailpiece id>`) works only
while the piece has not entered production. Once printed there is no recall.

## Pacing

Read `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset` from
every response — they are returned on all of them. The baseline is 300
requests/minute. Stannp documents no exhaustion status code and no `Retry-After`
header, so pace from `X-RateLimit-Reset` rather than waiting to be throttled.
