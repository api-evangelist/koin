---
name: Send a Koin payout by Pix or crypto
description: Resolve and validate the recipient account, quote a cryptocurrency where applicable,
  create the payout, and track it to Transferred or Failed.
api: openapi/koin-payments-openapi.json
operations:
  - RecipientAccountPOST
  - validateAccountPOST
  - getCurrenciesPOST
  - getQuotationsPOST
  - createPayoutPOST
  - payoutByPayoutId
  - payoutByReferenceIdGET
generated: '2026-07-19'
method: generated
source: openapi/koin-payments-openapi.json
---

# Send a Koin payout by Pix or crypto

Base URL (sandbox): `https://api-sandbox.koin.com.br`
Auth: `Authorization: Bearer sk_<32 alphanumeric>`.

Payouts are outbound transfers with their own lifecycle, separate from `PaymentOrder`:
`Published` → `Transferred` | `Failed`.

## Steps

1. **Resolve the recipient** — `POST /v1/payment/payment-key/info` (`RecipientAccountPOST`).
   Turns a payment key (Pix key) into recipient account information. Only `200`/`400` declared.

2. **Validate the account** — `POST /v1/payment/orders/{order_id}/account-validation`
   (`validateAccountPOST`) where the flow requires confirming the destination before transferring.

3. **For crypto only — pick a currency and quote it.**
   - `POST /v1/payment/payouts/currencies` (`getCurrenciesPOST`) returns the cryptocurrencies this
     account is permitted to transact. Do not assume a currency is enabled.
   - `POST /v1/payment/payouts/quotation` (`getQuotationsPOST`) returns the quote for the chosen
     currency. Quotes move — create the payout promptly against the quote you were given.

4. **Create the payout** — `POST /v1/payment/payouts` (`createPayoutPOST`).
   Send a stable unique `transaction.reference_id`. Required fields are validated first, then
   business rules. Supply `notification_url` to receive status callbacks.

5. **Track it** — `GET /v1/payment/payouts/{payout_id}` (`payoutByPayoutId`) or, by your own business
   key, `GET /v1/payment/payouts?reference_id=…` (`payoutByReferenceIdGET`).

## Webhook states

- `Published` — accepted and queued for transfer.
- `Transferred` — money moved. This is the success terminal state (not `Collected`, which is pay-in).
- `Failed` — carries `status.reason` (`Rejected`) and `status.message`.

Payout notifications use a distinct account suffix in Koin's own examples (`SBX000PO`) and carry
`payout_id` rather than `order_id`. Route them on payload shape, not on a shared handler.

## Money-movement cautions

A payout moves real funds and Koin exposes no idempotency header — the only duplicate protection is
your `reference_id`. Never retry a `createPayoutPOST` with a fresh `reference_id` after a timeout.
Re-issue the identical request, or call `payoutByReferenceIdGET` first to see whether the original
landed. Retry with exponential backoff on `5xx`/timeout only (integration requirement NOT1).

## Errors

`{code, message, causes[]}`; `400` validation, `404` unknown payout, `500` transient.
See `errors/koin-problem-types.yml` and `conventions/koin-conventions.yml`.
