# Docket Entry Tinfoil configuration

Minimal public deployment metadata for a synthetic infrastructure setup check. The image is Tinfoil's prebuilt hello-world example, pinned by digest. No application code, private prompts, credentials or case materials belong here.

The `v0.0.1` release and both release workflows completed successfully. A synthetic deployment passed the official Tinfoil SDK attestation check and returned HTTP 200 with the expected greeting. The container was last confirmed stopped, with its configuration retained. This checks release, deployment and attested connectivity only; it does not implement the Docket Entry Runner or demonstrate the project's M0 confidentiality proof.

This public repository remains deployment configuration for the synthetic setup check. The Docket Entry application repository remains private. No merge, rename or deletion is planned.

## Contents and provenance

- `tinfoil-config.yml`: the official example image and CVM version, using the minimum documented 2 CPUs and 8 GB memory. The greeting is generic and no secrets are requested.
- `.github/workflows/tinfoil-release.yml`: creates a version tag and dispatches publication.
- `.github/workflows/tinfoil-release-publish.yml`: measures the configuration, signs its attestation and publishes the GitHub release.

Both workflows are unchanged from [Tinfoil's official template at commit 0eddc320](https://github.com/tinfoilsh/tinfoil-containers-template/tree/0eddc320b8f328d7a3c057152596934444ac2d75). The config changes only the comment, container name and greeting, and removes the optional example secret declaration. The upstream action references and image are pinned.

## Release and deployment

The official **Tinfoil Release** workflow created version `v0.0.1`; the release and publication workflows both succeeded. The release was recognized and deployed for the synthetic check described above.

Keep real customer inputs, model credentials and proprietary prompts out of this example. Replace the example image with the reviewed Runner when it is ready; do not treat this greeting as completion of M0.

References: [quickstart](https://docs.tinfoil.sh/containers/quickstart), [configuration](https://docs.tinfoil.sh/containers/configuration), [private images](https://docs.tinfoil.sh/containers/private-images).
