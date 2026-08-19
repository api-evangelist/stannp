# Stannp Direct Mail API Reference

> LLM-optimised reference for the Stannp Direct Mail API. Base URL: `https://api-eu1.stannp.com` (EU/UK) or `https://api-us1.stannp.com` (US/CA). All endpoints below use the `/v1/` prefix.

## Authentication

Every request requires authentication. Two methods are supported:

1. **HTTP Basic Auth** -- use your API key as the username with an empty password: `-u {API_KEY}:`
2. **Query parameter** -- append `?api_key={API_KEY}` to the URL.

All requests **must** use HTTPS. Requests over HTTP will fail and may suspend your API key. You can find your API key at the bottom of your account settings page.

## Rate Limits

300 requests per minute for most endpoints. Every response includes these headers:

| Header | Description |
|--------|-------------|
| X-RateLimit-Limit | Maximum requests allowed per window |
| X-RateLimit-Remaining | Requests remaining in current window |
| X-RateLimit-Reset | Seconds until the rate limit window resets |

## Idempotency Keys

The API supports an optional `idempotency_key` parameter for safely retrying requests without performing a duplicate operation. If a duplicate idempotent request is detected, the original response body is returned with a `409` HTTP status. The full request body is compared, so only identical requests are treated as idempotent.

## Response Format

**Success:**
```json
{ "success": true, "data": { ... } }
```

**Error:**
```json
{ "success": false, "error": "Error message" }
```

**HTTP Status Codes:**

| Code | Meaning |
|------|---------|
| 200 OK | Successful result |
| 401 Unauthorized | Invalid or missing API key |
| 404 Not Found | Resource does not exist or was deleted |
| 409 Conflict | Idempotent request already processed (returns original response) |
| 500 Internal Server Error | Server-side error |

---

## Postcards

### POST /v1/postcards/create

Create and send a single postcard. Performs a mail merge to place the address and any variable data.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| size | string | optional | EU/UK: "A6", "A5", "A5-ENV", or "A5-PORT". Defaults to "A6". US: "4x6", "6x9", or "6x11". Defaults to "4x6". |
| test | boolean | optional | If true, produces a sample PDF without dispatching or charging. |
| template | int | optional | ID of a template already set up on the platform. Otherwise use front and back parameters. |
| recipient | mixed | mandatory | Either an ID of an existing recipient or a new recipient array. e.g., recipient[title], recipient[company], recipient[firstname], recipient[lastname], recipient[address1], recipient[address2], recipient[city], recipient[postcode], recipient[country] and recipient[*] for custom data. |
| message | string | optional | A message on the back of the card. If using a back image, this message will be overlaid on top. |
| signature | file | optional | An image placed in the signature location. Can be a URL, file, or base64 encoded string. Must be a JPG file with 768 x 118 pixels resolution. |
| front | file | optional | An image for the front. Can be a URL, file, or base64 encoded string. Required if no template is used. Supported: JPG, PNG, or PDF. |
| back | file | optional | An image for the back. Can be a URL, file, or base64 encoded string. Supported: JPG, PNG, or PDF. |
| padding | int | optional | A white border is added to the front by default. Set padding=0 to remove the border for edge-to-edge design. |
| post_unverified | boolean | optional | Default is true. If false, the item will not be posted if the recipient address could not be verified. |
| clearzone | boolean | optional | If true, overlays clear zones with a white background to keep the mailpiece machine-readable. Defaults to false. |
| return_address | string | optional | A custom return address to print on the postcard. |
| tags | string | optional | Comma-separated tags for your reference, searchable in reporting. |
| addons | string | optional | Addon codes to upgrade your postcard. Available: FIRST_CLASS (first-class postage), STOCK_350_SILK (350 GSM silk card), STOCK_350_UNCOATED (350 GSM uncoated card). Comma-separate multiple addons. |

**Response (test mode):**
```json
{
  "success": true,
  "data": {
    "pdf": "https://www.stannp.com/assets/samples/a6-postcard-sample.pdf",
    "id": 0,
    "created": "2024-01-15T10:30:00+00:00",
    "format": "A6",
    "cost": "0.88",
    "status": "test"
  }
}
```

In live mode, the response includes additional fields: type, title, firstname, lastname, company, address1-address3, city, county, postcode, country, addons, tags, tracking_ref, production_ref, dispatched, updated, transactional, and class.

---

### GET /v1/postcards/get/:id

Obtain the mailpiece object for the specified postcard ID.

**Parameters:** None (ID is in the URL path).

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 16818211,
    "timestamp": "2019-10-30 00:14:04",
    "status": "cancelled",
    "type": "postcard",
    "format": "A6",
    "pdf_file": "https://api-eu1.stannp.com/v1/storage/get/.../sample.pdf",
    "dispatched": null,
    "country": "GB",
    "cost": 0.00,
    "addons": "",
    "tags": "",
    "template_id": 0,
    "campaign_id": 0,
    "recipient_id": 0,
    "firstname": "John",
    "lastname": "Smith",
    "address1": "123 Sample Street",
    "address2": "",
    "address3": "",
    "city": "Sampletown",
    "county": "",
    "postcode": "AB12 3CD",
    "tracking_ref": null,
    "transactional": 0,
    "class": 0,
    "updated": "2019-10-30 00:14:04",
    "mailpiece_id": 16818211,
    "tracking": []
  }
}
```

---

### POST /v1/postcards/cancel

Cancel a postcard if processing has not yet started.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | The ID of the mailpiece item. |

**Response:**
```json
{ "success": true, "data": 1 }
```

---

## Letters

### POST /v1/letters/create

Create a single letter with mail merge for address and variable data.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| test | boolean | optional | If true, produces a sample PDF without dispatching or charging. |
| recipient | mixed | mandatory | Either an ID of an existing recipient or a new recipient array. e.g., recipient[title], recipient[company], recipient[firstname], recipient[lastname], recipient[address1], recipient[address2], recipient[city], recipient[postcode], recipient[country] and recipient[*] for custom data. |
| template | int | optional | ID of a template already set up on the platform. |
| file | file | optional | Alternative to template/pages -- send a PDF/DOC file directly. Maximum 10 pages. |
| size | string | optional | Paper format: "A4" or "US-LETTER". Defaults to "A4" for EU/UK accounts, "US-LETTER" for US accounts. |
| duplex | boolean | optional | Set to false for single-sided printing. Defaults to true. |
| clearzone | boolean | optional | If true, overlays clear zones with a white background. Defaults to true. |
| post_unverified | boolean | optional | Default is true. If false, the item will not be posted if the address could not be verified. |
| tags | string | optional | Comma-separated tags for your reference, searchable in reporting. |
| addons | string | optional | Addon codes: FIRST_CLASS (first-class postage), C4_ENVELOPE (C4 envelope), CONFIDENTIAL (confidential envelope), STOCK_120 (120 GSM paper). Comma-separate multiple addons. |

**Response (test mode):**
```json
{
  "success": true,
  "data": {
    "pdf": "https://www.stannp.com/assets/samples/letter-sample-12345.pdf",
    "id": 0,
    "created": "2024-01-15 10:30:00",
    "format": "A4",
    "cost": "0.81",
    "status": "test"
  }
}
```

In live mode, the response includes additional fields: type, title, firstname, lastname, company, address1-address3, city, county, postcode, country, addons, tags, tracking_ref, production_ref, dispatched, updated, transactional, and class.

---

### POST /v1/letters/post

Post a letter that already has the address on the PDF. The API uses OCR to extract the recipient address from the document.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| test | boolean | optional | If true, produces a sample PDF without dispatching or charging. |
| country | string | mandatory | ISO alpha-2 country code (e.g., US, CA, GB, FR, DE). |
| pdf | file | optional | A URL or binary PDF file to print and post. Also accepts "file" as parameter name. |
| size | string | optional | Paper format: "A4" or "US-LETTER". Defaults to "A4" for EU/UK, "US-LETTER" for US. |
| duplex | boolean | optional | Defaults to true. |
| transactional | boolean | optional | Use for sensitive data. Defaults to false. |
| tags | string | optional | Comma-separated tags for your reference, searchable in reporting. |
| addons | string | optional | Addon codes: FIRST_CLASS, C4_ENVELOPE, CONFIDENTIAL, STOCK_120. Comma-separate multiple addons. |

**Response (test mode):**
```json
{
  "success": true,
  "data": {
    "pdf": "https://www.stannp.com/assets/samples/letter-sample.pdf",
    "id": 0,
    "created": "2024-01-15 10:30:00",
    "format": "A4",
    "cost": "0.81",
    "status": "test"
  }
}
```

In live mode, the response includes additional fields: type, title, firstname, lastname, company, address1-address3, city, county, postcode, country, addons, tags, tracking_ref, production_ref, dispatched, updated, transactional, and class.

---

### GET /v1/letters/get/:id

Obtain the mailpiece object for the specified letter ID.

**Parameters:** None (ID is in the URL path).

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 16818210,
    "account_id": 5,
    "timestamp": "2019-02-13 00:14:04",
    "status": "cancelled",
    "type": "letter",
    "format": "A4",
    "pdf_file": "https://api-eu1.stannp.com/v1/storage/get/.../sample.pdf",
    "dispatched": null,
    "country": "GB",
    "cost": 0.00,
    "addons": "",
    "tags": "",
    "postcode": "AB12 3CD",
    "template_id": 0,
    "campaign_id": 0,
    "recipient_id": 0,
    "transactional": 0,
    "class": 0,
    "updated": "2019-02-13 00:14:04",
    "mailpiece_id": 16818210,
    "firstname": "John",
    "lastname": "Smith",
    "address1": "123 Sample Street",
    "address2": "",
    "address3": "",
    "city": "Sampletown",
    "county": "",
    "tracking_ref": null,
    "tracking": []
  }
}
```

---

### POST /v1/letters/cancel

Cancel a letter if processing has not yet started.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | The ID of the mailpiece item. |

**Response:**
```json
{ "success": true, "data": 1 }
```

---

## Recipients

### GET /v1/recipients/list

Get a list of recipient objects. Include group_id to filter by group. Supports pagination via offset and limit.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| group_id | int | optional | ID of the group to filter recipients. |
| offset | int | optional | Offset for pagination. |
| limit | int | optional | Limit for pagination. |

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "1943",
      "account_id": "5",
      "title": "Mr",
      "firstname": "John",
      "lastname": "Example",
      "company": "Example Co",
      "job_title": "",
      "address1": "123 Example street",
      "address2": "",
      "address3": "",
      "city": "Example Town",
      "county": "",
      "country": "GB",
      "postcode": "EX12 3AB",
      "dps": "",
      "email": "john@example.com",
      "phone_number": "+441234567890",
      "ref_id": "",
      "blacklist": "0",
      "created": "2024-01-15 10:30:00",
      "updated": "2024-01-15 10:30:00"
    }
  ]
}
```

---

### GET /v1/recipients/get/:id

Get a single recipient by ID.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | ID of the recipient to fetch. |

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "1943",
    "account_id": "5",
    "title": "Mr",
    "firstname": "John",
    "lastname": "Example",
    "company": "Example Co",
    "job_title": "",
    "address1": "123 Example street",
    "address2": "",
    "address3": "",
    "city": "Example Town",
    "county": "",
    "country": "GB",
    "postcode": "EX12 3AB",
    "dps": "",
    "email": "john@example.com",
    "phone_number": "+441234567890",
    "ref_id": "",
    "blacklist": "0",
    "created": "2024-01-15 10:30:00",
    "updated": "2024-01-15 10:30:00"
  }
}
```

---

### POST /v1/recipients/new

Create a new recipient.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| group_id | int | optional | The group ID to add the recipient to. |
| title | string | optional | Recipient's title (e.g., Mr, Mrs, Dr). |
| firstname | string | optional | Recipient's first name. |
| lastname | string | optional | Recipient's last name. |
| company | string | optional | Recipient's company name. |
| job_title | string | optional | Recipient's job title. |
| address1 | string | optional | Address line 1. |
| address2 | string | optional | Address line 2. |
| address3 | string | optional | Address line 3. |
| city | string | optional | Address city. |
| county | string | optional | County or state. Also accepts "state" as an alias. |
| postcode | string | optional | Address postal code. Also accepts "zipcode" as an alias. |
| country | string | optional | ISO 3166-1 Alpha 2 Country Code (GB, US, FR...). Defaults to account region if not provided. |
| email | string | optional | The recipient's email address. |
| phone_number | string | optional | The recipient's phone number. |
| ref_id | string | optional | Alternative ID for matching a recipient across multiple services. |
| on_duplicate | string | optional | What to do if a duplicate is found: update, ignore, or duplicate. |
| test_level | string | optional | How to test for duplicates. Defaults to "fullname". Options: "email" (match by email), "fullname" (match by full name and address), "initial" (match by initial of first + last name and address), "ref_id" (match by alternative ID). Custom fields can also be used as parameters. |

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "1941",
    "valid": true,
    "created": "2024-01-15T10:30:00+00:00"
  }
}
```

---

### POST /v1/recipients/delete

Permanently delete a recipient from your account.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | ID of the recipient to delete. |

**Response:**
```json
{ "success": true, "data": 1 }
```

---

### POST /v1/recipients/import

Import a data file into your account.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| file | file | mandatory | A CSV or XLS file as binary, a base64 encoded string of a CSV, or a URL to a file. |
| group_id | int | mandatory | ID of the group to import data into. |
| duplicates | string | optional | What to do if duplicate found: "update" (default, updates the record), "ignore" (do not add), "duplicate" (add all duplicates). |
| no_headings | boolean | optional | Set to true if your CSV has no heading row. If true, you must supply the mappings parameter. |
| mappings | string | optional | Comma-separated string to remap headings. e.g., "title,firstname,lastname,company,address1,address2,city,postcode,country,custom,custom,skip". Use "custom" to create a new field matching your heading name. Use "skip" to ignore a column. If not provided, headings must match: title, firstname, lastname, company, job_title, address1, address2, address3, city, postcode, country. |

**Response:**
```json
{ "success": true, "data": "1" }
```

---

### Block Listing Recipients

Recipients can be block listed via the Stannp platform. When a recipient is block listed, any sends to that recipient will be suppressed and not dispatched. The recipient object includes a `blacklist` field (boolean) that indicates whether the recipient is currently block listed. Block listing is useful for managing opt-outs and compliance with mailing preferences.

---

## Groups

### GET /v1/groups/list

Get a list of mailing groups on your account. Supports pagination.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| offset | int | optional | Offset for pagination. |
| limit | int | optional | Limit for pagination. |

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "398",
      "account_id": "1",
      "name": "Test Group",
      "created": "2015-09-25 11:57:35",
      "recipients": "100",
      "valid": "98",
      "international": "0",
      "skipped": "0",
      "status": "ready",
      "import_progress": "0",
      "is_seeds": "0"
    }
  ]
}
```

---

### POST /v1/groups/new

Create a new empty mailing list.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| name | string | mandatory | Name of the new group. |

**Response:**
```json
{ "success": true, "data": "39" }
```

---

### POST /v1/groups/add/:group_id

Add recipients to an existing mailing list.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| recipients | string | mandatory | Comma-separated recipient IDs. |

**Response:**
```json
{ "success": true, "data": 2 }
```

---

### POST /v1/groups/remove/:group_id

Remove recipients from a group. This only removes the recipient from the group and does not delete the recipient.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| recipients | string | mandatory | Comma-separated recipient IDs. |

**Response:**
```json
{ "success": true, "data": 2 }
```

---

### POST /v1/groups/purge

Remove all recipients from a mailing list.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | ID of the group to purge. |
| delete_recipients | boolean | optional | If true, completely deletes the recipients (they will not exist in any other groups). Defaults to false. |

**Response:**
```json
{ "success": true, "data": true }
```

---

### POST /v1/groups/calculate/:group_id

Recalculate a group to update its statistics.

**Parameters:** None (group_id is in the URL path).

**Response:**
```json
{ "success": true, "data": true }
```

---

### POST /v1/groups/delete

Delete a mailing list.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | ID of the group to delete. |
| delete_recipients | boolean | optional | If true, completely deletes the recipients. Defaults to false. WARNING: recipients will not exist in any other groups. |

**Response:**
```json
{ "success": true, "data": true }
```

---

## Campaigns

### GET /v1/campaigns/list

Get a list of your campaigns.

**Parameters:** None.

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "222",
      "account_id": "9",
      "name": "a5 test",
      "template_id": "0",
      "type": "a5-postcard",
      "trigger_date": null,
      "send_date": null,
      "trigger_offset": null,
      "created": "2015-04-26T10:40:00Z",
      "updated": "2015-04-26T10:40:00Z",
      "status": "draft",
      "recipients_group": "0",
      "recipients_filter": "",
      "recipients_validated": "valid",
      "recipients": "0",
      "recipients_not_valid": "0",
      "dispatched": "0",
      "cost": "0.00",
      "voucher_code": "",
      "image": null
    }
  ]
}
```

---

### GET /v1/campaigns/get/:id

Get details for a specific campaign.

**Parameters:** None (ID is in the URL path).

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "126",
    "name": "Stannp Test Campaign",
    "template_id": "111",
    "type": "a6-postcard",
    "trigger_date": null,
    "send_date": null,
    "trigger_offset": null,
    "created": "2015-03-17 11:04:19",
    "updated": "2015-03-21 09:10:05",
    "status": "dispatched",
    "recipients_group": "15",
    "recipients_filter": "",
    "recipients_validated": "valid",
    "recipients": "232",
    "recipients_not_valid": "0",
    "cost": "0.00",
    "voucher_code": "",
    "image": "https://dash.stannp.com/api/v1/uploads/9/image.webp",
    "size": "A6"
  }
}
```

---

### POST /v1/campaigns/create

Create a new campaign.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| name | string | mandatory | Name your campaign for reference. |
| type | string | mandatory | Campaign type: a6-postcard, a5-postcard, or letter. |
| template_id | int | optional | A template ID to copy. If 0 or omitted, a blank template of the correct format is used. |
| file | string | optional | A single or multi-page PDF file as URL or binary for design artwork. |
| front | string | optional | A PDF or JPG file for the front image (URL or binary). |
| back | string | optional | A PDF or JPG file for the back image (URL or binary). |
| size | string | optional | Required when using file or front/back images. e.g., "A4", "A5", "A6". |
| save | bool | optional | Save the uploaded file as a new design template on your account. |
| group_id | int | mandatory | A group ID for the recipient data. |
| what_recipients | string | optional | Which recipients to target: "all" (every recipient), "valid" (UK validated only), "not_valid" (UK non-validated only), "int" (international only). |
| addons | string | optional | Addon codes if applicable. |

**Response:**
```json
{ "success": true, "data": { "id": "266" } }
```

---

### POST /v1/campaigns/sample

Produce a PDF sample of your campaign. The link is valid for 30 minutes.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | The campaign ID to produce a sample for. |

**Response:**
```json
{
  "success": true,
  "data": "https://stannpstorage.blob.core.windows.net/pdf-samples/12345_sample.pdf"
}
```

---

### POST /v1/campaigns/approve

Approve a campaign. This locks the design and recipient selection. Once approved, changes cannot be made.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | The campaign ID to approve. |

**Response:**
```json
{ "success": true, "data": true }
```

---

### POST /v1/campaigns/cost

Get the cost breakdown and rates for a campaign.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | The campaign ID to get cost for. |

**Response:**
```json
{
  "success": true,
  "data": {
    "rate": "0.41",
    "non_valid_rate": "0.51",
    "delivery_rate": "0.00",
    "international_rate": "0.00",
    "valid": "14.35",
    "not_valid": "191.25",
    "international": "0.00",
    "delivery_charge": "0.00",
    "net": "205.60",
    "vat": "41.12",
    "total": "246.72"
  }
}
```

---

### POST /v1/campaigns/availableDates

Get available dates for booking a campaign.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| start | date | optional | Start date in YYYY-MM-DD format. Defaults to today. |
| end | date | optional | End date in YYYY-MM-DD format. Defaults to 30 days from today. |

**Response:**
```json
{
  "success": true,
  "data": ["2024-10-09", "2024-10-10", "2024-10-11", "..."]
}
```

---

### POST /v1/campaigns/book

Book a campaign for dispatch. Payment is taken from your balance.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | The campaign ID to book. |
| send_date | string | mandatory | The dispatch date in YYYY-MM-DD format. |
| next_available_date | bool | optional | Default is true. Books on the next available date if your chosen date is unavailable. Set to false to receive an error instead. |
| use_balance | bool | optional | Default is true. Use account balance to pay. If false, the campaign is scheduled but not posted until payment. |

**Response:**
```json
{ "success": true, "data": true }
```

---

### POST /v1/campaigns/delete

Delete a campaign. Only campaigns that have not been booked can be deleted.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | int | mandatory | The campaign ID to delete. |

**Response:**
```json
{ "success": true, "data": true }
```

---

## Events

### POST /v1/recipientEvents/create

Create a recipient event for tracking campaign engagement and conversions. Events can trigger automated communication.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| recipient_id | string | mandatory | The recipient ID or the alternative reference ID (ref_id) which can match an ID from a different system. |
| name | string | mandatory | Name the event. e.g., PURCHASE, SIGNUP, PAGE_VIEW, PRODUCT_VIEW, PRODUCT_TO_BASKET. |
| value | string | optional | Value information, e.g., the purchase amount or product name. |
| conversion | boolean | optional | Is this a conversion event (e.g., purchase or signup)? Defaults to false. |
| data | string | optional | Extended data for automation tasks or dynamic templating. |
| ref | string | optional | A campaign or mailpiece reference ID. If empty, the most recent communication is used as the reference. |

**Response:**
```json
{ "success": true, "data": 266 }
```

---

## Selections

### POST /v1/selections/new

Create an auto-filter selection for a group.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| group_id | int | mandatory | The ID of the group to associate this selection with. |
| name | string | optional | Name your filter. |
| filters | string | optional | Formatted filter string: `[column_name]:::[operator]:::[value]`. Operators: matches, contains, begins, ends, before, after, less_than, more_than, is_not. Chain multiple filters with `:::AND:::`. Example: `total_spent:::more_than:::300:::AND:::country:::matches:::GB` |

**Response:**
```json
{ "success": true, "data": [{ "id": "1234" }] }
```

---

## Account

### GET /v1/accounts/balance

Check your account balance.

**Parameters:** None.

**Response:**
```json
{ "success": true, "data": { "balance": "214.4200" } }
```

---

### POST /v1/accounts/topup

Top up your balance using a saved default card. Consider enabling auto top-up to ensure sends are not impacted by a low balance.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| net | string | mandatory | The amount to top up, e.g., "10.00". Tax may be added. |

**Response:**
```json
{
  "success": true,
  "data": {
    "receipt_pdf": "https://www.stannp.com/invoice/12345-receipt.pdf"
  }
}
```

---

## Addresses

### POST /v1/addresses/validate

Validate a postal address. Currently supports UK and US addresses.

**Parameters (EU/UK):**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| company | string | optional | Company name. |
| address1 | string | mandatory | Address line 1. |
| address2 | string | optional | Address line 2. |
| address3 | string | optional | Address line 3. |
| city | string | optional | Address city. |
| postcode | string | optional | Address postal code. Recommended for accurate validation. |
| country | string | optional | ISO 3166-1 Alpha 2 Country Code. Recommended for accurate validation. |

**Additional parameters (US):**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| state | string | optional | Two-letter state abbreviation. |
| zipcode | string | optional | Address zipcode. Required if city is not provided. |

**Additional parameters (CA):**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| province | string | optional | Two-letter province abbreviation (ON, BC, QC, etc.). |
| zipcode | string | optional | Address postal code. Required if city is not provided. |

**Response:**
```json
{
  "success": true,
  "data": {
    "address1": "Unit 12 Taw Trade Park",
    "address2": "",
    "address3": "",
    "city": "Barnstaple",
    "county": "Devon",
    "postcode": "EX31 1JZ",
    "country": "GB",
    "reference_id": 0,
    "dps": "1A",
    "udprn": "12345678",
    "is_valid": true
  }
}
```

---

## Reporting

### GET /v1/reporting/summary/:startdate/:enddate

Retrieve a status summary on individual items within a date range. Dates in YYYY-MM-DD format. The end date includes everything on that day.

**Parameters:** Dates are in the URL path.

**Response status fields:**

| Status | Description |
|--------|-------------|
| received | Request has been received. |
| producing | Mailpiece is being printed or finished. |
| handed_over | Handed over to the delivery service (Royal Mail, etc.). |
| local_delivery | At the final delivery office, will be delivered within 24 hours. |
| delivered | Estimated as successfully delivered. |
| cancelled | Cancelled before production and posting. |

**Response:**
```json
{
  "success": true,
  "data": {
    "total": 2000,
    "received": 0,
    "printing": 0,
    "handed_over": 0,
    "local_delivery": 400,
    "delivered": 1599,
    "returned": 1,
    "cancelled": 0
  }
}
```

---

### GET /v1/reporting/list/:startdate/:enddate/[:status]/[:tag]

Retrieve a list of mailpiece objects sent within a date range. Status and tag filters are optional.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| startdate | date | mandatory | Start date (YYYY-MM-DD). |
| enddate | date | mandatory | End date (YYYY-MM-DD). |
| status | string | optional | Status filter (e.g., delivered, returned). |
| tag | string | optional | Tag filter for searching by custom tags. |

**Response:**
```json
{
  "success": true,
  "data": {
    "singles": [
      {
        "id": "6829954",
        "timestamp": "2018-02-11 07:08:55",
        "status": "delivered",
        "type": "postcard",
        "format": "A6",
        "pdf_file": "https://dash.stannp.com/api/v1/storage/get/.../sample.pdf",
        "dispatched": "2018-02-11 15:12:52",
        "country": "GB",
        "cost": "0.42",
        "addons": "",
        "tags": "tag123,tag456",
        "postcode": "SW1A 1AA",
        "address": "10 Downing Street, London SW1A 2AA"
      }
    ]
  }
}
```

---

## Files

### POST /v1/files/upload

Upload a file to your secure file transfer area.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| file | string | mandatory | A binary file or a URL to a file. |
| folder | int | optional | A folder ID for placing the file in a folder. |

**Response:**
```json
{ "success": true, "data": { "id": "342" } }
```

---

### POST /v1/files/createFolder

Create a folder in your secure file transfer area.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| name | string | mandatory | The name of the folder to create. |

**Response:**
```json
{ "success": true, "data": "342" }
```

---

### GET /v1/files/folders

List folders in your secure file transfer area.

**Parameters:** None.

**Response:**
```json
{
  "success": true,
  "data": [
    { "id": "2", "name": "test folder", "created": "2017-06-27 14:24:07" },
    { "id": "15", "name": "test folder 2", "created": "2017-06-27 14:26:23" }
  ]
}
```

---

## SMS

### POST /v1/sms/create

Send an SMS message to a recipient's mobile device.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| test | boolean | optional | If true, the SMS will not be sent and no charge is taken. |
| phone_number | string | optional | The recipient's phone number. Required if recipient_id is not provided. |
| recipient_id | int | optional | ID of an existing recipient. Required if phone_number is not provided. |
| message | string | mandatory | The message to send. Supports template tags if using recipient_id (e.g., Hi {firstname}). |
| country | string | optional | 2-character country code (e.g., GB, US, CA). Defaults to account region. |

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 0,
    "recipient_id": 0,
    "phone_number": "01234567890",
    "from": "STANNP",
    "profile_id": 3,
    "usage_campaign_id": 0,
    "profile_type": "sender_id",
    "message": "Hello World",
    "message_parts": 1,
    "cost": 0.058,
    "destination_country": "GB",
    "status": "test",
    "campaign_id": 0,
    "created": "2024-07-25T16:16:28+00:00",
    "updated": ""
  }
}
```

---

## Tools

### GET|POST /v1/qrcode/create

Generate a QR code image. **Requires a PUBLIC KEY** (not a private API key). The `data` parameter accepts any content, typically a URL. Use template variables (e.g., `{{url}}`) for personalised QR codes per recipient during mail merge. Returns a JPEG image.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| size | int | optional | Requested size in pixels. Actual dimensions may differ as the QR code is sized to the nearest module boundary. |
| data | string | mandatory | The content the QR code should contain. |
| color | string | optional | Foreground colour as hex value. Defaults to #000000. |
| background | string | optional | Background colour as hex value. Defaults to #ffffff. |

**Response:** Binary JPEG image.

---

### POST /v1/pdf/merge

Merge multiple PDF files into a single file.

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| files | array | mandatory | An array of URLs for the PDF files to merge. Example: `files[]=https://file1.pdf&files[]=https://file2.pdf` |

**Response:**
```json
{ "success": true, "data": "https://merged-file.pdf" }
```

---

### GET|POST /v1/templates/list

Get the templates created on your account.

**Parameters:** None.

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "0",
      "account_id": "0",
      "template_name": "Template name 1",
      "image": "",
      "created": "2023-09-06 12:37:21",
      "updated": "2023-11-01 09:53:20",
      "size": "US-letter",
      "page_count": "1",
      "duplex": "1",
      "back_image": null,
      "template": "",
      "is_public": "1",
      "is_template": "1",
      "version": "3",
      "is_hidden": "1",
      "shared": "0",
      "user_id": "0",
      "tags": "",
      "versions": "3"
    }
  ]
}
```

---

## Objects

### Mailpiece Object

The object model of any mailpiece (postcard, letter, greeting card, etc.).

| Field | Description |
|-------|-------------|
| id | The mailpiece ID. |
| timestamp | Creation timestamp "YYYY-MM-DD HH:MM:SS". |
| status | The status of the mailpiece. See reporting section for status codes. |
| type | The mailpiece type. |
| format | The template format. |
| pdf_file | A URL of a PDF representation. |
| dispatched | Dispatched timestamp "YYYY-MM-DD HH:MM:SS". null if not yet dispatched. |
| cost | The unit cost of this item. |
| addons | Comma-separated list of addon codes. |
| tags | Reference tags for filtering and searching mailpieces. |

---

### Campaign Object

The object model of a campaign.

| Field | Description |
|-------|-------------|
| id | The campaign ID. |
| template_id | The template ID associated with the campaign. |
| status | Campaign status: draft, approved, provisioned, scheduled, running, paused, complete. |
| type | The campaign type. |
| created | Created timestamp ISO 8601 format. |
| updated | Updated timestamp ISO 8601 format. |
| send_date | The scheduled send date. ISO 8601 format. |
| dispatched | Dispatched timestamp ISO 8601 format. null if not yet dispatched. |
| cost | The cost of the campaign. |
| addons | Comma-separated list of addon codes. |
| image | Thumbnail image of the campaign after approval. |

---

### Recipient Object

The object model of a recipient.

| Field | Description |
|-------|-------------|
| id | The recipient ID. |
| email | Recipient email address. |
| title | Recipient title (e.g., Mr, Mrs). |
| firstname | First name. |
| lastname | Last name. |
| company | Recipient's company. |
| job_title | Recipient's job title. |
| address1 | Address line 1. |
| address2 | Address line 2. |
| address3 | Address line 3. |
| city | City or town. |
| county | County or state. |
| country | Country code in ISO 3166-1 alpha-2 format. |
| postcode | Postal code / zip code. |
| dps | Delivery point suffix. |
| ref_id | Reference ID for matching with another system. |
| blacklist | Whether the recipient has been block listed (boolean). |

---

## Webhooks

### Overview

Webhooks allow you to subscribe to events within the Stannp platform. When an event occurs, Stannp sends an HTTP POST request to your configured webhook URL.

### Creating a Webhook

Webhooks are created and managed in your Stannp account settings. On creation, a test request is sent to your URL looking for an HTTP 200 status. If not received, the webhook will not be created.

**Validation Request Body:**
```json
{
  "webhook_id": 0,
  "event": "test_url",
  "created": "2024-01-15 10:30:00",
  "retries": 0
}
```

### Webhook Payload

When triggered, the payload always contains an array of objects, even if just one item was triggered. The array key name varies by event type.

**Mailpiece Status payload:**
```json
{
  "webhook_id": 1234,
  "event": "mailpiece_status",
  "created": "2024-01-15 10:30:00",
  "retries": 0,
  "mailpieces": [{ "...": "[mailpiece object]" }]
}
```

**Campaign Status payload:**
```json
{
  "webhook_id": 1234,
  "event": "campaign_status",
  "created": "2024-01-15 10:30:00",
  "retries": 0,
  "campaigns": [{ "...": "[campaign object]" }]
}
```

**Recipient Events payload:**
```json
{
  "webhook_id": 1234,
  "event": "recipient_events",
  "created": "2024-01-15 10:30:00",
  "retries": 0,
  "events": [{ "...": "[event object]" }]
}
```

### Triggerable Events

| Event | Key | Description |
|-------|-----|-------------|
| campaign_status | campaigns | Triggers when campaign status changes. Statuses: printing, dispatched, cancelled. |
| mailpiece_status | mailpieces | Triggers when mailpiece status changes. Statuses: printing, dispatched, cancelled, local_delivery, delivered, returned. |
| recipient_events | events | Triggers when a new recipient event is recorded. |

### Webhook Security

Set a secret when creating your webhook to enable signed events. The signature is in the `X-Stannp-Signature` header, generated with an HMAC digest using SHA-256 of the raw request body against your secret.

To validate: recreate the hash by hashing the raw message body against your stored secret and compare with the `X-Stannp-Signature` header value. Use a proper hash comparator (e.g., PHP's `hash_equals()`) rather than `==`.

### Delivery Behaviour

- **Payload Chunking:** If an event triggers for more than 50 objects, the payload is split into multiple webhook calls, each containing up to 50 objects.
- **Retry Logic:** Failed deliveries are retried up to 3 times with a 30-second delay between attempts. The `retries` field reflects the current attempt number.
- **Auto-Pause:** If a webhook endpoint fails 5 or more times within 24 hours, the subscription is automatically paused. You must manually resubscribe via the platform.
