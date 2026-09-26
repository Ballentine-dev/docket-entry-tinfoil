# Docket Entry M1 Tinfoil configuration

This public repository contains configuration and documentation only. It does not change runtime or product code, the source repository, workflows, or API boundaries.

## Status and pinned artifacts

- **M0 is accepted.**
- **M1 remains incomplete; no M1 acceptance is claimed.** M1 v0.1.0 was published and deployed for a bounded synthetic test. Release verification, attested health, and role/release rejection checks passed. Storage returned an error, so encrypted persistence and restart validation remain incomplete.
- **M1 v0.1.1 is published and signed.** Its single measured live tracer invocation is recorded below. It did not establish storage success, and all four instances were stopped.
- **v0.1.2 is a config/docs-only draft, not published or live-tested.** It retains the existing images, networks, secrets, and other runtime configuration; only two entries are added to the buckets environment. No v0.1.2 pass is claimed.

The v0.1.1 Runner image is pinned to `ghcr.io/ballentine-dev/docket-entry-m0@sha256:1e444a3498c6a9353a67eea61d04530248f9851e1cc4f54ac05afefa8d7ecc72`, built from reviewed application commit `e8bcce3b84438b3dd00d8506d2e2ed42c0604537` ([build evidence](https://github.com/Ballentine-dev/docket-entry/actions/runs/36247915842)), runtime-identical to merged commit `72a5188`. The image build and all 250 offline tests passed for that Runner build. The proposed v0.1.2 keeps this exact Runner image; no application-code or image change is included.

The buckets sidecar is pinned to `ghcr.io/tinfoilsh/tinfoil-buckets-sidecar@sha256:03d43dd687a5ed352f3d4956db6af706ec9dee8ca2c0fbce651ee59317b2ede5`.

## Temporary storage tracer and measured result

The pinned Runner includes an issuer-only, read-only storage probe for a reserved synthetic path. It reports fixed outcomes, HTTP status, and timing; uses a separate in-memory probe key; and does not return response bodies or headers.

One live invocation in v0.1.1 reported:

- Sidecar: HTTP 400 in 131 ms.
- S3 leg: HTTP 500 in 8,781 ms.
- Health: `false`.
- All four instances were stopped.

No OAuth flow, model invocation, or persistent write occurred. No 404 was returned in this invocation. A 404 in a later probe would support the mitigation hypothesis; a fast 500 would suggest credential-delivery or client-initialization trouble; the observed slow S3-leg 500 leaves egress or reachability suspect. These are diagnostic observations, not conclusive proof of a specific root cause.

The `AWS_PROFILE` setting below is a temporary credential-delivery tracer and must be removed in the first post-diagnosis release. The proposed v0.1.2 leaves the Runner image and storage probe unchanged.

## Proposed buckets diagnostic settings

The unpublished v0.1.2 draft adds exactly these two environment settings to `buckets`:

- `JAVA_TOOL_OPTIONS=-Djava.net.preferIPv4Stack=true` asks Java to prefer the IPv4 stack; it is a diagnostic setting, not a demonstrated fix.
- `AWS_PROFILE=cred-delivery-probe` selects a profile-provider path when the static credential pair is unavailable.

According to `Config.resolveCreds` in the pinned official sidecar v0.0.6, non-null `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` values select static credentials. Otherwise, a non-empty `AWS_PROFILE` selects the profile provider; the default credential chain is used only when neither condition applies. Thus, if either explicit credential value is null, this setting selects the `cred-delivery-probe` profile path rather than the default-chain branch. If both values are non-null, static credentials still take precedence. An unavailable profile may itself cause an error: the profile name supplies no credentials and does not prove credential delivery. No profile contents or credentials are included in this repository.

## Scope and behavior

The configured `EMAIL` release is for synthetic evaluation only, not production or real customer data. The model budget is $0.05 per job, with a per-increment ceiling of <=$1. These are stated budget limits/targets, not measured spend or proof of runtime enforcement.

Intended reboot behavior is that access to existing encrypted objects remains possible with the same customer-held key, without repeating OAuth. This behavior remains pending live verification; the measured tests did not complete storage validation. In multitenant mode, the customer supplies the key with requests. No operator encryption key or plaintext AWS credentials are included in this repository.

M2 in-flight recovery is not claimed. The sidecar keeps multipart session state locally, so an interrupted multipart session does not survive a sidecar restart.

## Topology and storage notes

The shim forwards only `/health` and `/v1/*` to `runner:8080`. The buckets service listens on port 9000 on the shared internal network; it is not exposed through the shim. Runner egress is allowlisted to `openrouter.ai`; buckets egress is allowlisted to the two S3 hostnames in the config. Each container has one egress-enabled network plus the egress-closed internal network.

Only the buckets container references the Tinfoil secrets `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. `MULTITENANT=true` means there is no operator `ENCRYPTION_KEY`; the sidecar expects tenant and customer-held encryption-key headers per request. The sidecar does not authenticate requests itself and trusts those headers, so keep it internal and do not expose it directly.

The sidecar is configured for buffered GETs with `BUFFER_SIZE=1048576` (1 MiB) and `DANGEROUS_DELAYED_AUTH=false`. GETs larger than the buffer are rejected rather than streamed. The sidecar also requires path-style S3 requests, does not support ranged GETs, and requires sequential multipart uploads with non-final parts aligned to 16 bytes.

No host port mappings or writable mounts are declared. The config relies on the platform's default read-only filesystem; both pinned containers reached Running without restarts or OOM in the first test, but storage functionality remains unverified.
