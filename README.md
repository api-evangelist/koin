# Koin

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
