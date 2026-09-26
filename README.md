# Docket Entry M1 Tinfoil configuration

This public repository contains configuration and documentation only. It does not change runtime or product code, the source repository, workflows, or API boundaries.

## Status and pinned artifacts

- **M0 is accepted.**
- **M1 v0.1.0 is published and was deployed for a bounded synthetic test.** Release verification, attested health and role/release rejection checks passed. Storage returned an error, so encrypted persistence and restart validation remain incomplete. The test instance is stopped; no M1 acceptance is claimed.
- The buckets sidecar is pinned to `ghcr.io/tinfoilsh/tinfoil-buckets-sidecar@sha256:03d43dd687a5ed352f3d4956db6af706ec9dee8ca2c0fbce651ee59317b2ede5`.

The next diagnostic release pins Runner `sha256:1e444a3498c6a9353a67eea61d04530248f9851e1cc4f54ac05afefa8d7ecc72`, built from reviewed application commit `e8bcce3b84438b3dd00d8506d2e2ed42c0604537` ([build evidence](https://github.com/Ballentine-dev/docket-entry/actions/runs/36247915842)), runtime-identical to merged commit `72a5188`. All 250 offline tests and the image build passed. This adds an issuer-only, read-only storage probe returning fixed outcomes, HTTP status and timing for a reserved synthetic path. It uses a separate in-memory probe key and does not return bodies or headers. A new measured release and actual probe result are still required; it is not a storage fix or M1 acceptance. The sidecar/configuration boundaries remain the same.

## Scope and behavior

The configured `EMAIL` release is for synthetic evaluation only, not production or real customer data. The model budget is $0.05 per job, with a per-increment ceiling of <=$1. These are stated budget limits/targets, not measured spend or proof of runtime enforcement.

Intended reboot behavior is that access to existing encrypted objects remains possible with the same customer-held key, without repeating OAuth. This behavior remains pending live verification; the first deployment did not complete storage validation. In multitenant mode, the customer supplies the key with requests. No operator encryption key or plaintext AWS credentials are included in this repository.

M2 in-flight recovery is not claimed. The sidecar keeps multipart session state locally, so an interrupted multipart session does not survive a sidecar restart.

## Topology and storage notes

The shim forwards only `/health` and `/v1/*` to `runner:8080`. The buckets service listens on port 9000 on the shared internal network; it is not exposed through the shim. Runner egress is allowlisted to `openrouter.ai`; buckets egress is allowlisted to the two S3 hostnames in the config. Each container has one egress-enabled network plus the egress-closed internal network.

Only the buckets container references the Tinfoil secrets `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. `MULTITENANT=true` means there is no operator `ENCRYPTION_KEY`; the sidecar expects tenant and customer-held encryption-key headers per request. The sidecar does not authenticate requests itself and trusts those headers, so keep it internal and do not expose it directly.

The sidecar is configured for buffered GETs with `BUFFER_SIZE=1048576` (1 MiB) and `DANGEROUS_DELAYED_AUTH=false`. GETs larger than the buffer are rejected rather than streamed. The sidecar also requires path-style S3 requests, does not support ranged GETs, and requires sequential multipart uploads with non-final parts aligned to 16 bytes.

No host port mappings or writable mounts are declared. The config relies on the platform's default read-only filesystem; both pinned containers reached Running without restarts or OOM in the first test, but storage functionality remains unverified.
