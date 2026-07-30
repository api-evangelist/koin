---
name: Offer Koin BNPL installments at checkout
description: Check whether a buyer is eligible for buy-now-pay-later, present the returned installment
  options, create the BNPL order, and follow it to a terminal state through webhooks.
api: openapi/koin-payments-openapi.json
operations:
  - availabilityPOST
  - createPayment
  - paymentByReferenceIdGET
  - cancelPaymentPUT
generated: '2026-07-19'
method: generated
source: openapi/koin-payments-openapi.json
---

# Offer Koin BNPL installments at checkout

Base URL (sandbox): `https://api-sandbox.koin.com.br`
Auth: `Authorization: Bearer sk_<32 alphanumeric>`.

Koin's BNPL ("Pix Parcelado" / installment boleto) lets a Brazilian buyer pay over time with no card
and no pre-registration. Two integration shapes exist: **transparent** (you render the options) and
**redirect** (Koin hosts the selection page). Availability is transparent-checkout only.

## Steps

1. **Check eligibility** — `POST /v1/payment/orders/availability` (`availabilityPOST`).
   Returns the installment options for this buyer and basket on success. Only `200` and `400` are
   declared. If the buyer is not eligible, fall through to another payment method rather than
   attempting the order.

2. **Present the options.** Each option carries `installments`, `installment_rate`,
   `installment_amount`, `total_amount`, `currency_code` and `first_due_date`. Show the rate — Koin
   documents simple vs compound interest explicitly and expects merchants to render it honestly.

3. **Create the BNPL order** — `POST /v1/payment/orders` (`createPayment`) with the BNPL payment
   method and the selected installment option. Supply `notification_url`.
   - In the **redirect** flow the response (and the `Waiting` notification) carries `return_url`
     pointing at `https://payments.koin.com.br/checkout/{order_uuid}`. Send the buyer there; they
     return to you afterwards.
   - Use a stable unique `reference_id`. A `409`, or business code `511`/`998`, means Koin already has
     this order — look it up, do not re-create.

4. **Wait for the decision.** BNPL is asynchronous. Do not fulfil on the create response.
   - `Waiting` with reason `FirstPayment`, `EmailValidation` or `WhatsAppValidation` — the buyer still
     has an action to complete.
   - `Pending` with reason `ProviderReview` — under review.
   - `Collected` — approved and collected. Fulfil.
   - `Cancelled` — do not fulfil.

5. **Cancel if needed** — `PUT /v1/payment/orders/{order_id}/cancel` (`cancelPaymentPUT`).

6. **Look up by business key at any time** — `GET /v1/payment/orders?reference_id=…`
   (`paymentByReferenceIdGET`).

## Decision codes

BNPL orders carry Koin's numeric business codes alongside the status. `200` approved; `300`/`302`
declined (offer another method); `701` exceeds the buyer's Koin credit limit; `702` credit decline;
`312` awaiting e-mail activation or under review; `703` discount value not allowed; `10101`/`10103`
CPF or e-mail already registered. Full registry with the exact buyer-facing wording:
`errors/koin-decline-codes.yml`. Never invent your own decline copy — Koin publishes the strings.

## Sandbox

Force the outcome with the buyer e-mail: `john_autoaccept@test.com` approves,
`john_autoreject@test.com` declines, `john_autoinprogress@test.com` holds the order open,
`john_manualaccept@test.com` routes through manual review. See `sandbox/koin-sandbox.yml`.

## Legacy contract

An older BNPL Payment Request API exists on `https://www.sp-api.koin.com` — mint a token with
`generate-by-rest`, then `check` eligibility and `include` the payment request. Prefer the `/v1`
Payments contract above for new integrations.
