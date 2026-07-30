---
name: Charge a card with Koin anti-fraud
description: Tokenize a card, run a Koin anti-fraud pre-evaluation before authorization, create the
  card payment, then capture it and reconcile the result from the webhook.
api: openapi/koin-payments-openapi.json
operations:
  - tokenizeCardPOST
  - createPreEvaluationUsingPOST
  - createPayment
  - capturePayment
  - paymentByOrderIdGET
  - createEvaluationUsingPOST
generated: '2026-07-19'
method: generated
source: openapi/koin-payments-openapi.json, openapi/koin-antifraud-evaluations-openapi.json
---

# Charge a card with Koin anti-fraud

Base URL (sandbox): `https://api-sandbox.koin.com.br`
Auth: `Authorization: Bearer sk_<32 alphanumeric>` — one private key covers payments, antifraud and BNPL.
Always send `Content-Type: application/json` and `Accept: application/json`; both are declared required.

## Before you start

- Mint a stable, unique `reference_id` for this order and keep it. It is Koin's idempotency key
  (integration requirement COR1): reuse the SAME value across pre-evaluation, evaluation and any
  later lookup. Never generate a second one for a retry.
- Collect the device fingerprint first. Koin requires its JavaScript snippet on the page where the
  payer starts or completes payment; the identifier it returns goes in the `device` object. An
  evaluation without `device` will not pass certification.

## Steps

1. **Tokenize the card** — `POST /v1/payment/tokenize` (`tokenizeCardPOST`).
   Send the card object; receive a `secure_token`. Raw PAN never touches your servers if you use the
   browser Checkout SDK instead. Only `200` and `400` are declared — a `400` means field validation
   failed; read `causes[]`.

2. **Pre-evaluate the risk** — `POST /v1/antifraud/pre-evaluations` (`createPreEvaluationUsingPOST`).
   Run this BEFORE acquirer authorization. Pass `store.reference_id` = your `reference_id` and the
   `device` object. A rejection here means you should not attempt authorization at all.

3. **Create the payment** — `POST /v1/payment/orders` (`createPayment`).
   Reference the `secure_token` from step 1 and supply `notification_url` with your HTTPS callback
   endpoints — this is the only way to subscribe to webhooks; there is no registration API.
   - `409 Conflict` means Koin already accepted this `reference_id`. Do NOT re-create. Call
     `paymentByReferenceIdGET` instead.
   - Business codes `511` and `998` also signal a duplicate submission.

4. **Full evaluation after authorization** — `POST /v1/antifraud/evaluations`
   (`createEvaluationUsingPOST`), reusing the same `reference_id`. This is mandatory when the product
   requires post-authorization analysis.

5. **Capture** — `POST /v1/payment/orders/{order_id}/capture` (`capturePayment`) when the payment was
   authorized rather than auto-captured. Only `200`, `404`, `500` are declared.

6. **Reconcile** — read state with `GET /v1/payment/orders/{order_id}` (`paymentByOrderIdGET`), or by
   business key with `GET /v1/payment/orders?reference_id=…` (`paymentByReferenceIdGET`).

## Handling the result

Webhooks are the source of truth for terminal state. Card events: `Authorized`, `Collected`,
`Voided`, `Cancelled`, `Refunded`, `Failed` (reason `Rejected`). Persist the callback BEFORE returning
`2xx`, and process replays idempotently — the same event must never cause a second capture or a
duplicate order release.

## Errors

Koin does not use RFC 9457. Errors are `application/json` with `{code, message, causes[]}`.
`400` validation (read `causes[]`), `401` bad private key, `404` unknown order, `409` duplicate,
`500` transient — retry with exponential backoff, never with a new `reference_id`.

Business decision codes (`200` approved, `300`/`302`/`701`/`702` declined, `312` under review) are in
`errors/koin-decline-codes.yml`. Decline reasons are deliberately masked from the buyer.

## Sandbox

Force outcomes with buyer e-mail patterns: `john_autoaccept@test.com`, `john_autoreject@test.com`,
`john_preaccept_autoreject@test.com`, `john_auto_inprogress_3ds2_autoaccept@test.com`.
3DS test cards (exp `12`/`2030`, CVV `876`): `4000000000001000` frictionless approved,
`4000000000002503` challenge approved, `4000000000002644` challenge failure,
`5200000000001013` frictionless failure. See `sandbox/koin-sandbox.yml`.
