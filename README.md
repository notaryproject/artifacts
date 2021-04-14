# OCI Artifacts (Experimental) with OCI Artifact Reference Support

This is an experimental fork of oci artifacts, validating the [oci.artifact.manifest spec][oci-artifact-manifest] support of reference types. Reference types are required to meet the [Notary v2 Requirements][nv2-requirements] for not changing the target digest or tag, and the [Notary v2 Scenarios][nv2-scenarios] for content movement within and across registry implementations.

![](media/net-monitor-graph.svg)

## Table of Contents:

- [OCI Artifact Manifest Overview](./artifact-manifest.md)
- [OCI Artifact Manifest Spec](./artifact-manifest-spec.md)
- [ORAS experimental support for oci.artifact.manifest references][oras-artifacts] to `push`, `discover`, `pull` referenced artifact types.

[oci-artifact-manifest]:  https://github.com/SteveLasker/artifacts/blob/oci-artifact-manifest/artifact-manifest.md
[nv2-requirements]:       https://github.com/notaryproject/notaryproject/blob/main/requirements.md
[nv2-scenarios]:          https://github.com/notaryproject/notaryproject/blob/main/scenarios.md
[oras-artifacts]:         https://github.com/deislabs/oras/blob/prototype-2/docs/artifact-manifest.md