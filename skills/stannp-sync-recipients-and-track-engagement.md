---
name: Sync recipients into Stannp and track engagement
description: >-
  Keep a CRM's contacts in sync with Stannp recipient groups using your own
  reference IDs, then close the loop by posting conversion events back so mail
  can be attributed and automations triggered.
api: openapi/stannp-recipients-api-openapi.yml
operations:
  - createGroup
  - createRecipient
  - importRecipients
  - listRecipients
  - getRecipient
  - deleteRecipient
  - addRecipientsToGroup
  - removeRecipientsFromGroup
  - purgeGroup
  - createRecipientEvent
generated: '2026-08-13'
method: generated
source: https://www.stannp.com/us/direct-mail-api/recipients
---

# Sync recipients into Stannp and track engagement

This is the most common Stannp integration: push contacts in, send mail against
them, push conversions back.

## 1. Anchor on your own identifier

Set `ref_id` on every recipient you create. It is the field designed for
cross-system matching, and `createRecipientEvent` accepts a `ref_id` where it
would otherwise want a Stannp `recipient_id`. Without it you must store Stannp's
integer IDs yourself; Stannp IDs carry no prefix and cannot be routed by
inspection.

## 2. Create or update

`createRecipient` (`POST /v1/recipients/new`) with `group_id` and the address
fields. Control duplicate handling explicitly:

- `on_duplicate` = `update` | `ignore` | `duplicate`
- `test_level` = `email` | `fullname` (default) | `initial` | `ref_id`

Set `test_level=ref_id` when your own identifier is authoritative — otherwise
Stannp matches on name and address and two genuinely different people at one
address can collapse into one record.

The response is `{"success": true, "data": {"id": "1941", "valid": true, ...}}`.
`valid` is the address-verification verdict, and it is the field to gate on
before sending: an unverified address either does not deliver or is charged at
the higher non-validated rate.

For bulk, use `importRecipients` (`POST /v1/recipients/import`) with `file` +
`group_id`, plus `mappings` when your CSV headings do not match Stannp's
(`title,firstname,lastname,company,address1,address2,city,postcode,country,custom,skip`
— `custom` creates a new field from your heading, `skip` drops the column).
Import is asynchronous; watch `import_progress` on the group.

## 3. Read back

`listRecipients` (`GET /v1/recipients/list`) takes `group_id`, `offset` and
`limit`. There is **no total, no cursor and no has-more field** — page until a
short or empty array comes back. `getRecipient` (`GET /v1/recipients/get/{id}`)
fetches one.

Check `blacklist` on the recipient object. A block-listed recipient is suppressed
at send time and mail will not be dispatched to them; that is how opt-outs are
honoured, so mirror the flag back into your CRM rather than re-adding the contact.

## 4. Move contacts between groups

- `addRecipientsToGroup` (`POST /v1/groups/add/{group_id}`) and
  `removeRecipientsFromGroup` (`POST /v1/groups/remove/{group_id}`) both take
  `recipients=<comma-separated IDs>`. Removing from a group does **not** delete
  the recipient.
- `purgeGroup` and `deleteGroup` accept `delete_recipients=true`, which
  **permanently deletes the recipient records across every group**. So does
  `deleteRecipient`. These are destructive beyond the scope they appear to
  operate on — never issue them from an automated sync without an explicit,
  scoped instruction.

## 5. Close the loop with events

`createRecipientEvent` (`POST /v1/recipientEvents/create`):

| Field | Notes |
|---|---|
| `recipient_id` | Stannp ID **or** your `ref_id` — mandatory |
| `name` | e.g. `PURCHASE`, `SIGNUP`, `PAGE_VIEW`, `PRODUCT_VIEW`, `PRODUCT_TO_BASKET` — mandatory |
| `value` | purchase amount, product name, etc. |
| `conversion` | `true` for a purchase or signup; defaults to false |
| `data` | extended JSON for automations and dynamic templating |
| `ref` | campaign or mailpiece reference; omit and Stannp attributes to the most recent communication |

Set `ref` explicitly when you know which mailpiece drove the conversion.
Attribution to "the most recent communication" is a guess, and it is the guess
your ROI reporting will be built on.

Events you post are echoed back on the `recipient_events` webhook, which is how a
downstream system learns about conversions recorded by a sibling service.

## Conventions that apply throughout

- Form-encoded bodies only; bracket notation for nested fields.
- The response `data` field is polymorphic — an object, an array, a bare integer
  or a bare string depending on the operation.
- Errors are `{"success": false, "error": "<free text>"}` with no error code.
- Read `X-RateLimit-Remaining` and `X-RateLimit-Reset` on every response; the
  baseline is 300 req/min and no exhaustion status code is documented.
- A bulk sync is the workload most likely to hit the limit — pace imports rather
  than looping `createRecipient`.
