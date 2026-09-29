# Docket Entry Tinfoil configuration

This public repository contains configuration and documentation only. It does not change runtime or product code, the source repository, workflows, or API boundaries.

## Current configuration (v0.3.1)

`tinfoil-config.yml` is the source of truth. If this README and the config disagree, the config wins. Every fact in this section is read from the config at tag `v0.3.1`, except the reason given for open storage egress.

- **Workload:** `m2-v0.3.1`. The Runner is in `m2` mode with the `development` environment. It carries the `EMAIL`, `MAJOR_STANDARD`, `MAJOR_ADVANCED` and `MAJOR_MAX` releases, all with release id `m2-v0.3.1`.
- **Routes:** the shim forwards `/health`, `/v1/*` and `/v2/*` to `runner:8080`.
- **Runner egress:** allowlisted to `openrouter.ai`.
- **Storage egress:** open. This is a temporary, vendor-directed workaround for tinfoilsh/cvmimage#379. It is to be flipped back when the vendor fix ships.
- **Internal network:** egress closed. Each container is on it, plus one other network: `provider` for the Runner and `storage` for the buckets sidecar.
- **Runner image:** `ghcr.io/ballentine-dev/docket-entry-m0@sha256:299a7bbf7d826bc8bcf39bd25977db8a8b17ccc757b7d3e6939f9da5cf6f19ea`.
- **Buckets sidecar image:** `ghcr.io/tinfoilsh/tinfoil-buckets-sidecar@sha256:03d43dd687a5ed352f3d4956db6af706ec9dee8ca2c0fbce651ee59317b2ede5`.
- **Buckets sidecar:** listens on port 9000 on the internal network and is not reachable through the shim. `MULTITENANT` is `true`. Only this container references the Tinfoil secrets `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
- **CVM:** `cvm-version: "0.14.7"`, `cpus: 2`, `memory: 8192`.

## Historical M1 and M2 evidence

Everything below records earlier tags and is kept as evidence. It is not the current configuration. Status and behavior statements describe the tag they were written for. Where anything below differs from the section above or from `tinfoil-config.yml`, those win.

M1 Path A was accepted with temporary open storage egress and the approved full-reboot customer-key restoration pause.

### M2 v0.2.0 measured release candidate

This candidate switches to the reviewed M2 recovery runtime and its public configuration, and adds `/v2/*` to the existing health/connection routes. The private application passed 860 tests and required PR CI before merge. The image digest is updated only from the verified build of that merge. The CVM version, resources, sidecar image/settings, OpenRouter-only provider egress, open storage workaround and closed internal network remain unchanged. No raw credentials or customer material are published.

The synthetic proof uses two actual Shopify dev-store test-card orders and their actual signed, filtered webhooks. One entitlement must survive Runner kill/restart and finish on the same job; a separate entitlement must exhaust 14 releases without confirmation and raise the refund flag. Synthetic research credential custody is checked in the same increment. Live proof and founder M2 acceptance remain pending. Deploy only one instance, verify the signed exact-tag measurement, and stop the instance after the bounded proof. Path B and research fulfillment remain separate.

### M1 status and pinned artifacts

- **M0 is accepted.**
- **M1 remains incomplete; no M1 acceptance is claimed.** M1 v0.1.0 was published and deployed for a bounded synthetic test. Release verification, attested health, and role/release rejection checks passed. Storage returned an error, so encrypted persistence and restart validation remain incomplete.
- **M1 v0.1.1 is published and signed.** Its single measured live tracer invocation reported sidecar HTTP 400 in 131 ms, S3-leg HTTP 500 in 8,781 ms, and health `false`. All four instances were stopped.
- **v0.1.2 is published and signed but was never deployed.** The diagnostic specification changed after tagging; the revised combined DNS/config diagnostic was run under v0.1.3.
- **v0.1.3 has one verified live issuer-only probe, detailed below.** The exact tag, config, signature, and measurement were verified. The Runner image used for v0.1.3 and retained here is `ghcr.io/ballentine-dev/docket-entry-m0@sha256:684d1d3fe41ad51caee195467299650a301aa7f12bbec9fadf1bfda0aa605731`, built from reviewed application commit `6cc21ec7558b25418cfb77e651bacdaf8f3a9f43`, runtime-identical to merged `4714133`. All 267 tests, required PR checks, final-main CI, and image build passed for v0.1.3.
- **v0.1.4 is tagged.** The only config change from the v0.1.3 configuration is removal of the `AWS_PROFILE` environment entry from `buckets`. The pinned images, networks, secret references, IPv4 preference, issuer/DNS probe, and all other settings are unchanged. No v0.1.4 live measurement is claimed.

The buckets sidecar digest is the one pinned in the current configuration above; it has not changed since v0.1.3.

### v0.1.3 issuer-only probe and DNS observations

The pinned Runner includes an issuer-only, read-only storage probe for a reserved synthetic path. It reports fixed outcomes, HTTP status, and timing; uses a separate in-memory probe key; and returns no response bodies or headers. It also emits bounded DNS observations for the two S3 hostnames configured at v0.1.3. The DNS results below are from the Runner network namespace; they do not establish the sidecar's or egress enforcer's DNS answers. DNS failures do not change HTTP-derived storage health.

One real issuer-only invocation in v0.1.3 reported:

- Buckets sidecar: HTTP 400 in 131 ms.
- S3 leg: HTTP 404 in 929 ms.
- Health: `true`.
- `s3.us-east-2.amazonaws.com`: eight IPv4 A records, TTL 4 seconds.
- `docket-entry-m1-212346651704-us-east-2.s3.us-east-2.amazonaws.com`: CNAME to `s3-r-w.us-east-2.amazonaws.com` with TTL 300 seconds, followed by eight A records with TTL 5 seconds.

No OAuth flow, model call, or persistent write occurred. Instance shutdown was initiated immediately after the run, and all five instances were confirmed stopped. These observations do not establish a specific root cause or verify encrypted persistence and restart behavior.

### Buckets diagnostic settings

- `JAVA_TOOL_OPTIONS=-Djava.net.preferIPv4Stack=true` remains configured and was present during the v0.1.3 probe. That run did not isolate this setting's causal effect; it is retained configuration, not a demonstrated fix.
- The temporary `AWS_PROFILE` credential-delivery tracer used for v0.1.3 has been removed from the `buckets` environment in v0.1.4. The issuer-only storage and DNS probe remains in the pinned Runner.

### Scope and behavior

Through v0.2.1 the configuration carried a single `EMAIL` release. It is for synthetic evaluation only, not production or real customer data. The model budget is $0.05 per job, with a per-increment ceiling of <=$1. These are stated budget limits/targets, not measured spend or proof of runtime enforcement.

Intended reboot behavior is that access to existing encrypted objects remains possible with the same customer-held key, without repeating OAuth. This remains pending live verification; the v0.1.3 probe made no persistent write and did not test restart behavior. In multitenant mode, the customer supplies the key with requests. No operator encryption key or plaintext AWS credentials are included in this repository.

The bounded follow-up is a repeat health probe plus real encrypted persistence and restart tests; these remain pending.

The M2 recovery runtime is configured from v0.2.0 (`DOCKET_RUNNER_MODE` is `m2`); the v0.1.x text here predates it. The sidecar keeps multipart session state locally, so an interrupted multipart session does not survive a sidecar restart.

### Topology and storage notes

From v0.1.0 through v0.1.6 the shim forwarded only `/health` and `/v1/*` to `runner:8080`. From v0.1.0 through v0.1.5 buckets egress was allowlisted to S3 hostnames: two through v0.1.4, three at v0.1.5. Storage egress has been open since v0.1.6.

`MULTITENANT=true` means there is no operator `ENCRYPTION_KEY`; the sidecar expects tenant and customer-held encryption-key headers per request. The sidecar does not authenticate requests itself and trusts those headers, so keep it internal and do not expose it directly.

The sidecar is configured for buffered GETs with `BUFFER_SIZE=1048576` (1 MiB) and `DANGEROUS_DELAYED_AUTH=false`. GETs larger than the buffer are rejected rather than streamed. The sidecar also requires path-style S3 requests, does not support ranged GETs, and requires sequential multipart uploads with non-final parts aligned to 16 bytes.

No host port mappings or writable mounts are declared. The config relies on the platform's default read-only filesystem; both pinned containers reached Running without restarts or OOM in the first test, but encrypted persistence and restart behavior remain unverified.
