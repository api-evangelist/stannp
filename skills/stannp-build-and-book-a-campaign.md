---
name: Build, price and book a Stannp campaign
description: >-
  Run a batch direct mail campaign end to end — build a recipient group, create
  the campaign, proof it, price it before committing spend, approve it and book
  a dispatch date.
api: openapi/stannp-campaigns-api-openapi.yml
operations:
  - createGroup
  - createRecipient
  - importRecipients
  - recalculateGroup
  - createCampaign
  - getCampaignSample
  - getCampaignCost
  - approveCampaign
  - getCampaignAvailableDates
  - bookCampaign
  - getCampaign
  - deleteCampaign
generated: '2026-08-13'
method: generated
source: https://www.stannp.com/us/direct-mail-api/campaigns
---

# Build, price and book a Stannp campaign

A campaign is a batch send: one recipient group, one design, one dispatch date.
It has an approval gate and a payment step, and both are one-way.

## 1. Build the group

1. `createGroup` (`POST /v1/groups/new`, `name=<group name>`) → returns the new
   group ID as `data`.
2. Add recipients either one at a time with `createRecipient`
   (`POST /v1/recipients/new`, with `group_id` and the address fields) or in bulk
   with `importRecipients` (`POST /v1/recipients/import`, `file` + `group_id`).
   `file` accepts a binary CSV/XLS, a base64 CSV string, or a URL.
   - Set `duplicates=update|ignore|duplicate` on import, or `on_duplicate` plus
     `test_level=email|fullname|initial|ref_id` on a single create, to control
     de-duplication.
   - Set `ref_id` to your own system's identifier so you can join back later —
     `recipientEvents/create` accepts a `ref_id` in place of a Stannp ID.
3. `recalculateGroup` (`POST /v1/groups/calculate/{group_id}`) refreshes the
   derived counters. Then `listGroups` (`GET /v1/groups/list`) to read
   `recipients`, `valid`, `international` and `skipped` — these do not update on
   write, so recalculate before you trust them.

Import is asynchronous. Watch `import_progress` and `status` on the group before
building a campaign against it.

## 2. Create the campaign

`createCampaign` (`POST /v1/campaigns/create`) requires `name`, `type`
(`a6-postcard`, `a5-postcard` or `letter`) and `group_id`. Supply the design as
`template_id`, or as `file` / `front` / `back` URLs with an explicit `size`.

`what_recipients` selects who is targeted: `all`, `valid` (validated addresses
only), `not_valid`, or `int` (international only). It defaults to targeting
everyone — set it deliberately.

Returns `{"success": true, "data": {"id": "266"}}`.

## 3. Proof it

`getCampaignSample` (`POST /v1/campaigns/sample`, `id=<campaign id>`) renders a
full proof PDF. **The returned link expires after 30 minutes** — fetch and store
the bytes if a human needs to review it later.

## 4. Price it before you commit

`getCampaignCost` (`POST /v1/campaigns/cost`, `id=<campaign id>`) returns the
complete breakdown *without* spending anything:

```
rate, non_valid_rate, delivery_rate, international_rate,
valid, not_valid, international, delivery_charge, net, vat, total
```

This is the dry run. Compare `total` against the account balance from
`getAccountBalance` before going further. If a human or a spend policy needs to
approve the send, this is the point to stop and ask.

## 5. Approve — one way

`approveCampaign` (`POST /v1/campaigns/approve`, `id=<campaign id>`) locks the
design and the recipient selection. **Approval cannot be undone.** Do not call it
on behalf of a human who has not seen the sample PDF and the cost breakdown.

## 6. Book a date

1. `getCampaignAvailableDates` (`POST /v1/campaigns/availableDates`, optional
   `start` / `end` as `YYYY-MM-DD`) → an array of bookable dates.
2. `bookCampaign` (`POST /v1/campaigns/book`) with `id` and
   `send_date=YYYY-MM-DD`.
   - `next_available_date` defaults to **true**, which silently rolls your send
     to a later date if the one you asked for is unavailable. Set it to `false`
     if you would rather receive an error than have the date changed under you.
   - `use_balance` defaults to **true** and takes payment from the account
     balance immediately. Set it to `false` to schedule without paying; the
     campaign will not post until it is paid.

Booking is the point of no return for spend.

## 7. Afterwards

- `getCampaign` (`GET /v1/campaigns/get/{id}`) for status: `draft` → `approved` →
  `provisioned` → `scheduled` → `running` → `complete` (also `paused`,
  `cancelled`).
- `deleteCampaign` (`POST /v1/campaigns/delete`) works **only** on campaigns that
  have not been booked.
- Subscribe to the `campaign_status` webhook (`printing`, `dispatched`,
  `cancelled`) instead of polling. Deliveries chunk at 50 objects per call and
  auto-pause after 5 failures in 24 hours, with no API to resume — see
  `asyncapi/stannp-webhooks.yml`.

## Agent guardrails

The irreversible steps are **approve**, **book** and any group operation with
`delete_recipients=true` (which deletes the recipient records everywhere, not
just from this group). Treat all three as human-in-the-loop. See
`agentic-access/stannp-agentic-access.yml`.
