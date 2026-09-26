# Docket Entry M1 Tinfoil configuration

This public repository contains configuration and documentation only. It does not change runtime or product code, the source repository, workflows, or API boundaries.

## Status and pinned artifacts

- **M0 is accepted.**
- **M1 v0.1.0 is prepared but not published.** Live M1 deployment and validation are pending; this configuration is not evidence of a successful deployment.
- The runner image is pinned to `ghcr.io/ballentine-dev/docket-entry-m0@sha256:42cf026ff772687a1d044d60aa9547b3df8ff2c288eed4fb38bf5f96f6471e26`, built from app commit `1fc534e756719c7064ed7ff8acb53457d8ae1d2d`. App commit `c9e1608` changes tooling only; this config retains the supplied runtime image.
- The buckets sidecar is pinned to `ghcr.io/tinfoilsh/tinfoil-buckets-sidecar@sha256:03d43dd687a5ed352f3d4956db6af706ec9dee8ca2c0fbce651ee59317b2ede5`.

The Runner image is pinned to `sha256:87f6ec5226ec26febb7cc1395fca9dc7e3c6837f4ccb1494420a71dc9d1e57be`, built from reviewed application commit `fa3910c74155e00d6a6815b4a4c120c11b4ead17` ([build evidence](https://github.com/Ballentine-dev/docket-entry/actions/runs/36243887475)). The runtime content is identical to merged application commit `914bd40`. All 226 offline tests and the image build passed. This includes the reviewed interrupted-storage-response correction. The sidecar is pinned to the official v0.0.6 digest in the configuration.

## Scope and behavior

The configured `EMAIL` release is for synthetic evaluation only, not production or real customer data. The model budget is $0.05 per job, with a per-increment ceiling of <=$1. These are stated budget limits/targets, not measured spend or proof of runtime enforcement.

Intended reboot behavior is that access to existing encrypted objects remains possible with the same customer-held key, without repeating OAuth. This behavior is pending live verification; M1 has not yet been deployed and validated. In multitenant mode, the customer supplies the key with requests. No operator encryption key or plaintext AWS credentials are included in this repository.

M2 in-flight recovery is not claimed. The sidecar keeps multipart session state locally, so an interrupted multipart session does not survive a sidecar restart.

## Topology and storage notes

The shim forwards only `/health` and `/v1/*` to `runner:8080`. The buckets service listens on port 9000 on the shared internal network; it is not exposed through the shim. Runner egress is allowlisted to `openrouter.ai`; buckets egress is allowlisted to the two S3 hostnames in the config. Each container has one egress-enabled network plus the egress-closed internal network.

Only the buckets container references the Tinfoil secrets `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. `MULTITENANT=true` means there is no operator `ENCRYPTION_KEY`; the sidecar expects tenant and customer-held encryption-key headers per request. The sidecar does not authenticate requests itself and trusts those headers, so keep it internal and do not expose it directly.

The sidecar is configured for buffered GETs with `BUFFER_SIZE=1048576` (1 MiB) and `DANGEROUS_DELAYED_AUTH=false`. GETs larger than the buffer are rejected rather than streamed. The sidecar also requires path-style S3 requests, does not support ranged GETs, and requires sequential multipart uploads with non-final parts aligned to 16 bytes.

No host port mappings or writable mounts are declared. The config relies on the platform's default read-only filesystem; compatibility with these pinned images has not been validated.
