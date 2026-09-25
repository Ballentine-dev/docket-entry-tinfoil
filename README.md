# Docket Entry Tinfoil configuration

Public deployment metadata for the synthetic M0 Runner. Application source remains in the private Docket Entry repository. No private prompts, credentials or case materials belong here.

The prepared configuration replaces the original hello-world image with the reviewed Runner image, pinned by digest. It exposes only health and job routes, uses 2 CPUs and 8 GB memory, and permits outbound requests only to `openrouter.ai`. No secrets or persistent volumes are configured.

## Evidence and limits

Release `v0.0.1` previously passed the official SDK attestation and greeting check and was stopped afterward. That was infrastructure evidence only.

This Runner image was built from application commit `f292ceee0b81e57ac5ccda5cd5f978536969a236` after 30 offline tests, review, refactoring and a successful CI container build. A new measured release and real execution test remain to be completed. Configuration publication alone does not prove execution or pass M0.

The synthetic client supplies customer and instruction inputs separately over verified channels. The prototype uses in-memory jobs and manual protected test-key intake; production identity, browser OAuth and persistence are not established.

## Release and deployment

The two release workflows retain Tinfoil's official pinned actions from [template commit 0eddc320](https://github.com/tinfoilsh/tinfoil-containers-template/tree/0eddc320b8f328d7a3c057152596934444ac2d75). **Tinfoil Release** creates a version tag and dispatches measurement, signing and publication. Deploy the intended tag with debug disabled, and require the exact approved code measurement in the test client.

The image must be accessible to Tinfoil before deployment. Its digest identifies the reviewed code; it does not grant registry access. Keep this synthetic package separate from any future private production image.

References: [configuration](https://docs.tinfoil.sh/containers/configuration), [networking](https://docs.tinfoil.sh/containers/config-networking), [private images](https://docs.tinfoil.sh/containers/private-images).
