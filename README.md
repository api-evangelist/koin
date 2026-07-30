# Koin

Koin is a Brazilian payments and fraud-prevention provider operating across Latin America. It gives
merchants a single private-key API covering BNPL ("Pix Parcelado" — installment payments with no
card), instant Pix pay-in and payout, card payments with anti-fraud, crypto payouts, refunds,
cancellations and account validation. Its Antifraud API scores e-commerce, wire-transfer and
account-takeover events with mandatory device fingerprinting, 3-D Secure 2 and strategy-based
recovery.

Founded in Brazil, backed by Speedinvest (2018), acquired by Despegar in 2020.

- Website: https://www.koin.com.br/
- Developer portal: https://api-docs.koin.com.br/
- Platform integration docs: https://docs.koin.com.br/
- GitHub: https://github.com/koinlatam
- Status: https://koin.statuspage.io/

## APIs

| API | Operations | Contract |
|---|---|---|
| Payments API | 18 | `openapi/koin-payments-openapi.json` |
| Antifraud API (5 contracts) | 10 | `openapi/koin-antifraud-*-openapi.json` |
| Onboarding API | 2 | `openapi/koin-onboarding-openapi.json` |
| BNPL Payment Request API (legacy) | 3 | `openapi/koin-bnpl-openapi.json` |

## Artifacts

| Artifact | Path |
|---|---|
| OpenAPI (8 specs, 33 operations) | `openapi/` |
| Overlays (our enhancements) | `overlays/` |
| Authentication | `authentication/koin-authentication.yml` |
| Conventions + idempotency | `conventions/koin-conventions.yml` |
| Error catalog | `errors/koin-problem-types.yml` |
| Decline / business codes | `errors/koin-decline-codes.yml` |
| Webhook catalog | `asyncapi/koin-payments-webhooks.yml` |
| Sandbox + test values | `sandbox/koin-sandbox.yml` |
| Lifecycle + status page | `lifecycle/koin-lifecycle.yml` |
| Changelog | `changelog/koin-changelog.yml` |
| Packages / SDKs | `packages/koin-packages.yml` |
| Embedded components | `components/koin-components.yml` |
| Data model | `data-model/koin-data-model.yml` |
| Conformance | `conformance/koin-conformance.yml` |
| MCP (candidate) | `mcp/koin-mcp.yml` |
| Agent Skills | `skills/` |
| llms.txt (verbatim) | `llms/koin-llms.txt` |
| Domain security | `security/koin-domain-security.yml` |
| Well-known (verified absent) | `well-known/koin-well-known.yml` |

## Notes

- Authentication is a single bearer private key (`sk_` + 32 characters) covering payments, antifraud
  and BNPL. There is no OAuth surface, so no `scopes/` artifact is produced.
- Idempotency is reference-id based, not header based, and Koin makes replay-safe processing a
  contractual integration requirement.
- Koin publishes no AsyncAPI, no `/.well-known/` documents, no security.txt, no vulnerability
  disclosure program, no trust center and no deprecation policy. These are recorded as verified
  absences rather than gaps.
