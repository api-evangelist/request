---
name: Create and send a crypto/fiat invoice
description: Issue an invoice through the Request Finance AP/AR API and track it to paid.
api: https://api.request.finance/
operations:
  - POST /invoices
  - GET /invoices/{id}
  - POST /invoices/{id}/changes
generated: '2026-07-20'
method: generated
source: https://docs.request.finance/invoices
---

# Create and send a crypto/fiat invoice

Use this skill to issue an invoice and follow it through to payment.

## Auth & headers
- Production: OAuth 2.0 / OIDC Bearer access token (`Authorization: Bearer <token>`), plus `X-Network: live` (or `test`). See `authentication/request-authentication.yml`.
- Quick start only: a Test/live API key in the `Authorization` header (deprecated — prefer OAuth).
- Always send `Accept: application/json` and `Content-Type: application/json`.
- Test mode persists invoices on the Sepolia testnet; explore them at `https://baguette-app.request.finance/` (see `sandbox/request-sandbox.yml`).

## Steps
1. **Create the invoice** — `POST /invoices` with `meta` (`format: rnf_invoice`, `version`), a unique `invoiceNumber`, `invoiceItems[]` (`unitPrice`, `quantity`, `currency`), `paymentOptions[]` (recipient address + accepted currencies), `paymentTerms.dueDate`, `buyerInfo`, and `sellerInfo`. Use a currency from `GET /currency/list/invoicing`. Response returns the invoice `id` and `requestId`.
2. **Fetch / confirm** — `GET /invoices/{id}` (add `?withLinks=true` for share and payment links).
3. **Lifecycle changes** — `POST /invoices/{id}/changes` with `type`: `accept` (buyer approves), `reject` (buyer rejects — requires `input.note`), or `cancel` (seller voids). Only `open` invoices can be approved; `open`/`accepted` can be rejected or canceled.
4. **Mark externally paid** — `POST /invoices/{id}/changes` with `type: declareReceivedPayment` and `input.amount` (+ `input.txLink` for crypto). Regular on-platform payments are detected automatically.

## Conventions & idempotency
- `invoiceNumber` must be unique per invoice — it is the natural dedup key on create.
- Standard HTTP status codes; JSON bodies. Rate-limit headers `X-Rate-Limit-*` and `Retry-After` on 429 (see `conventions/request-conventions.yml`).

## Track via webhooks
Subscribe to `create`/`accept`/`reject`/`cancel`/`declareReceivedPayment`/`paid` events (verify `X-Webhook-Signature`, HMAC-SHA256). See `asyncapi/request-webhooks.yml`.
