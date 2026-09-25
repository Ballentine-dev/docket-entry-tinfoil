# Docket Entry Tinfoil configuration

Public deployment metadata for the synthetic M0 Runner. Application source remains in the private Docket Entry repository. No private prompts, credentials or case materials belong here.

The configuration replaces the original hello-world image with the reviewed Runner image, pinned by digest. It exposes only health and job routes, uses 2 CPUs and 8 GB memory, and permits outbound requests only to `openrouter.ai`. No secrets or persistent volumes are configured.

## Evidence and limits

Release `v0.0.1` previously passed the official SDK attestation and greeting check and was stopped afterward. That was infrastructure evidence only.

This Runner image was built from application commit `f292ceee0b81e57ac5ccda5cd5f978536969a236` after 30 offline tests, review, refactoring and a successful CI container build. Only the synthetic image package was made public with founder approval; the application repository remains private.

[Release v0.0.2](https://github.com/Ballentine-dev/docket-entry-tinfoil/releases/tag/v0.0.2) passed signature and configuration verification and deployed with debug disabled. On September 25, 2026, the official SDK's attested TLS transport verified both input channels against the exact approved release. Four real model calls completed and returned the expected synthetic result, 23. Provider billing metadata reconciled all four calls at $0.0001623 total. A deliberately wrong release measurement was rejected before credential read. The test instance was confirmed stopped afterward; both instances are stopped. The billing dashboard reported $0.02 container spend for the current period, not a final per-run invoice.

The synthetic client supplies customer and instruction inputs separately over verified channels. The prototype uses in-memory jobs and manual protected test-key intake; production identity, browser OAuth and persistence are not established. Full M0 remains pending browser authorization and the remaining real authorization/failure/leak checks. Detailed sanitized receipts and handoff are retained in the private application repository.

## Release and deployment

The two release workflows retain Tinfoil's official pinned actions from [template commit 0eddc320](https://github.com/tinfoilsh/tinfoil-containers-template/tree/0eddc320b8f328d7a3c057152596934444ac2d75). **Tinfoil Release** creates a version tag and dispatches measurement, signing and publication. Deploy the intended tag with debug disabled, and require the exact approved code measurement in the test client.

The image must be accessible to Tinfoil before deployment. Its digest identifies the reviewed code; it does not grant registry access. Keep this synthetic package separate from any future private production image.

References: [configuration](https://docs.tinfoil.sh/containers/configuration), [networking](https://docs.tinfoil.sh/containers/config-networking), [private images](https://docs.tinfoil.sh/containers/private-images).
