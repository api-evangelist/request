---
name: Run a payroll payment
description: Pay an employee or contractor in crypto or fiat via the Request Finance payroll API.
api: https://api.request.finance/
operations:
  - POST /invoices
  - GET /invoices
generated: '2026-07-20'
method: generated
source: https://docs.request.finance/salaries
---

# Run a payroll payment

Payroll payments are invoices with `meta.format: rnf_salary`. They are created on the same `POST /invoices` endpoint.

## Auth & headers
- OAuth 2.0 / OIDC Bearer token + `X-Network: test|live` (see `authentication/request-authentication.yml`).
- `Accept: application/json`, `Content-Type: application/json`.

## Steps
1. **Create the payroll payment** — `POST /invoices` with:
   - `meta`: `{ format: "rnf_salary", version: "0.0.3" }`
   - unique `invoiceNumber`
   - `invoiceItems[]`: `unitPrice` (payroll amount), `quantity` (`1`), `currency` (denomination, e.g. USD), optional `name` (e.g. "Salary - February").
   - `paymentOptions[]`: recipient address + payout currency (the employee can be paid in crypto even if denominated in fiat). Validate against `GET /currency/list/invoicing`.
   - `sellerInfo` (employee: `email` required, `firstName`, `lastName`) and `buyerInfo.email` (employer).
   - optional `paymentTerms.dueDate` and `recurringRule` (iCal RFC 5545, e.g. `RRULE:FREQ=MONTHLY;INTERVAL=1`) for recurring salaries.
2. **List payroll payments** — `GET /invoices?variant=rnf_salary` (offset pagination via `take`/`skip`, max `take=100`).
3. **Track to paid** — watch webhook events (`create` → `paid`; crypto-to-fiat also emits `declareSentPayment`/`declareReceivedPayment`). Verify `X-Webhook-Signature`.

## Testing
Use a Test API key / `X-Network: test`; pay on Sepolia with ETH (faucet) or FAU (DAI-pegged testnet token). The issuing account cannot pay itself — create a second account for the payer. See `sandbox/request-sandbox.yml`.
