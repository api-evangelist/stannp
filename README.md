# Stannp (stannp)

Stannp is a direct mail platform that enables businesses to send physical postcards and letters programmatically via a REST API. The platform lets developers create campaigns, upload recipient lists, trigger individual mail pieces in real time, and track print and delivery status through webhooks and event endpoints. Authentication uses API key-based HTTP Basic Auth over HTTPS, and the API follows a simple JSON response envelope. Stannp serves businesses across the UK, US, and Canada with per-item pricing for letters and postcards at scale, and supports no-code integrations through Zapier and Make as well as official SDKs for PHP, Go, and C#.

**APIs.json:** https://raw.githubusercontent.com/api-evangelist/stannp/refs/heads/main/apis.yml

**Naftiko:** https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=stannp-api-evangelist&utm_content=repo

## Tags

- Direct Mail
- Postcards
- Letters
- Print
- Physical Mail
- Marketing Automation
- Campaigns

## APIs

| Name | Description | Human URL |
|------|-------------|-----------|
| Stannp Direct Mail API | REST API for programmatically sending physical letters and postcards, managing recipient groups, configuring campaigns, triggering individual mail pieces, and tracking delivery status through webhooks and event callbacks. | https://www.stannp.com/us/direct-mail-api |

## Plans / Rate Limits / FinOps

| Resource | Path |
|----------|------|
| Plans & Pricing | [plans/stannp-plans-pricing.yml](plans/stannp-plans-pricing.yml) |
| Rate Limits | [rate-limits/stannp-rate-limits.yml](rate-limits/stannp-rate-limits.yml) |
| FinOps | [finops/stannp-finops.yml](finops/stannp-finops.yml) |

**Pricing model:** Freemium — Free, Starter (£12/mo), Growth (£48/mo), Premium (£315/mo), Enterprise (custom). Annual billing saves 20%. Per-item metered charges for each letter or postcard dispatched apply on top of the subscription.

**Rate limits:** 300 req/min (Free/Starter), 600 req/min (Growth), 2000 req/min (Premium), 3000 req/min (Enterprise). 429 status on breach; Retry-After header provided.

## Timestamps

- **Created:** 2026-06-12
- **Modified:** 2026-06-12

## Common Properties

| Type | URL |
|------|-----|
| Website | https://www.stannp.com |
| Documentation | https://www.stannp.com/us/direct-mail-api/guide |
| GitHub Organization | https://github.com/Stannp |
| LinkedIn | https://www.linkedin.com/company/stannp-com-postcard-bulk-mailer |
| X (Twitter) | https://twitter.com/stannpdm |
| Blog | https://go.stannp.com/blogs |
| Pricing | https://www.stannp.com/us/pricing-tiers |

## Maintainers

| Name | Email |
|------|-------|
| Kin Lane | kin@apievangelist.com |
