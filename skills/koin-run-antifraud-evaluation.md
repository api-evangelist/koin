---
name: Run a Koin anti-fraud evaluation
description: Score a transaction for fraud — pre-evaluation before authorization, full evaluation
  after, then poll status, push notifications back to Koin, or cancel the evaluation.
api: openapi/koin-antifraud-evaluations-openapi.json
operations:
  - createPreEvaluationUsingPOST
  - createEvaluationUsingPOST
  - evaluationStatusUsingGET
  - sendEvaluationUpdatesUsingPATCH
  - cancelEvaluationUsingDELETE
  - healthCheckUsingGET
  - createWireTransferUsingPOST
  - atoEvaluationUsingPOST
generated: '2026-07-19'
method: generated
source: openapi/koin-antifraud-evaluations-openapi.json, openapi/koin-antifraud-lifecycle-openapi.json,
  openapi/koin-antifraud-wire-transfer-openapi.json, openapi/koin-antifraud-ato-openapi.json
---

# Run a Koin anti-fraud evaluation

Base URL (sandbox): `https://api-sandbox.koin.com.br`
Auth: `Authorization: Bearer sk_<32 alphanumeric>`.

Koin's Antifraud API can be used standalone — you keep your own acquirer and use Koin purely for the
risk decision.

## The reference_id rule

One stable, unique `reference_id` per business transaction, REUSED between pre-evaluation and full
evaluation (integration requirement COR1). This is what ties the two calls into one case. Getting
this wrong produces two unrelated evaluations and a wrong decision.

## Device fingerprint is mandatory

Embed Koin's JavaScript snippet on the pages where the payer starts or completes payment, let it run
before you submit, and pass the token/session it returns in the `device` object. Substituting another
library requires written authorization from Koin. Native apps use the Android or iOS fingerprinting
SDK (`packages/koin-packages.yml`). An evaluation missing `device` will fail certification.

## Steps

1. **Pre-evaluate** — `POST /v1/antifraud/pre-evaluations` (`createPreEvaluationUsingPOST`).
   Run this BEFORE sending the transaction to the acquirer. A rejection means do not authorize.

2. **Authorize** at your acquirer if the pre-evaluation allowed it.

3. **Full evaluation** — `POST /v1/antifraud/evaluations` (`createEvaluationUsingPOST`) with the SAME
   `reference_id`. Returns the evaluation id and decision.

4. **Poll status** — `GET /v1/antifraud/evaluations/{id}` (`evaluationStatusUsingGET`) for cases held
   open for background verification.

5. **Report back** — `PATCH /v1/antifraud/notifications/{id}` (`sendEvaluationUpdatesUsingPATCH`).
   Notification types: `RFI` (request for information), `Chargeback`, `Status` (`Collected`,
   `Not Collected`, `Cancelled`, `Finalized`, `Refunded`, `Authorized`, `Recovering`), and `Info`.
   Feeding outcomes back is what keeps Koin's model calibrated.
   Address the case by your own key with `?field=REFERENCE_ID` instead of the Koin evaluation id.

6. **Cancel** — `DELETE /v1/antifraud/evaluations/{id}` (`cancelEvaluationUsingDELETE`) when the store
   or the buyer abandons the order before analysis completes. The `{id}` may be the Koin evaluation
   id or your `store.referenceId`.

7. **Health** — `GET /v1/antifraud/healthCheck` (`healthCheckUsingGET`).

## Other evaluation shapes

- **Wire transfer** — `POST /v1/antifraud/evaluations` (`createWireTransferUsingPOST`), a distinct
  request body on the same path.
- **Account takeover** — `POST /v1/ato/evaluations` (`atoEvaluationUsingPOST`) scores an ATO attempt;
  update it with `PATCH /v1/ato/notifications/{id}` (`sendAccountTakeOverUpdatesUsingPATCH`).

## Strategies and 3DS

When a decision is recoverable Koin may apply a **strategy** — verification code, liveness check,
manual review, authentication recovery. Koin may extend the type list, so treat the Strategies
reference and the live OpenAPI as source of truth rather than hard-coding an enum.

For 3DS2, only send the acquirer pre-authorization AFTER 3DS completes, forwarding `additional_info`
(CAVV, ECI, XID, `directory_server_transaction_id`, spec version) unchanged so liability shift
applies.

## Errors

`{code, message, causes[]}` — `400` validation (e.g. `causes: ["device.ip must not be blank"]`),
`401` bad private key, `404` unknown evaluation, `500` transient. See `errors/koin-problem-types.yml`.

## Sandbox

Force the decision with the buyer e-mail: `_prereject_` rejects at authorization,
`_preaccept_autoreject_` accepts then rejects at evaluation, `_autoinprogress_` holds open,
`_manualreject_` routes through manual review, `_auto_inprogress_3ds2_autoaccept_` exercises 3DS.
See `sandbox/koin-sandbox.yml`.
