---
name: Collect a Pix payment and refund it
description: Create a Pix pay-in, reconcile settlement from the webhook using the end-to-end id, then
  issue a full or partial refund and confirm it.
api: openapi/koin-payments-openapi.json
operations:
  - createPayment
  - paymentByOrderIdGET
  - createRefundPUTbyOrderId
  - consultRefundGET
  - cancelPaymentPUT
generated: '2026-07-19'
method: generated
source: openapi/koin-payments-openapi.json
---

# Collect a Pix payment and refund it

Base URL (sandbox): `https://api-sandbox.koin.com.br`
Auth: `Authorization: Bearer sk_<32 alphanumeric>`.

## Steps

1. **Create the Pix order** — `POST /v1/payment/orders` (`createPayment`) with a Pix payment method.
   Koin returns the QR code / copy-and-paste payload for the buyer. Supply `notification_url`.
   Use a stable unique `reference_id`; a `409` means Koin already accepted this order — call
   `paymentByReferenceIdGET` rather than creating a second one.

2. **Wait for settlement.** Pix is asynchronous — never fulfil on the create response. The
   `Collected` webhook carries `transaction.provider_reference.tx_id` and `end_to_end_id`, plus
   `provider.code` (for example `ITAU`). Persist both: `end_to_end_id` is the Pix scheme's
   reconciliation key. Terminal Pix states are `Collected`, `Refunded`, `Cancelled`, `Failed`.

3. **Read current state on demand** — `GET /v1/payment/orders/{order_id}` (`paymentByOrderIdGET`).

4. **Refund** — `PUT /v1/payment/orders/{order_id}/refund` (`createRefundPUTbyOrderId`).
   May be full or partial; send the amount to return. Returns a `refund_id`.

5. **Confirm the refund** — `GET /v1/payment/orders/{order_id}/refunds/{refund_id}`
   (`consultRefundGET`). Only `200`, `404`, `500` are declared. The `Refunded` webhook independently
   carries `refund_id` and `refund_amount`.

## Cancel vs refund vs void

- **Cancel** (`cancelPaymentPUT`, `PUT /v1/payment/orders/{order_id}/cancel`) — for Pix, only
  **dynamic QR** payments can be cancelled. For card, cancelling an `Opened` payment leaves it
  `Cancelled`; cancelling an `Authorized` payment leaves it `Voided`.
- **Refund** — returns money on an already-approved payment, full or partial.

Pick the right one: a settled Pix cannot be cancelled, only refunded.

## Errors

`{code, message, causes[]}` in `application/json` — not RFC 9457. `400` validation,
`401` bad key, `404` unknown order/refund, `500` transient (retry with backoff, same `reference_id`).
See `errors/koin-problem-types.yml`.

## Reconciliation rules

Process webhook replays idempotently (integration requirements INF4/CBK3) — a repeated `Collected`
must not release the order twice. Persist the callback before returning `2xx`, and serve the callback
URL over TLS.
