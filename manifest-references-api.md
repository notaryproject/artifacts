# Manifest References API

[OCI artifact-manifest](./artifact-manifest.md) provides the ability to reference artifacts to existing artifacts. Reference artifacts include Notary v2 signatures, SBoMs Nydus expansion images and many other types. Artifacts that reference others SHOULD not be tagged, as they are considered enhancements to the artifacts they reference. To discover referenced artifacts a manifest references API is provided. An artifact client, such as a Notary v2 client would parse the returned manifests, determining which manifest type they will pull and process.

The `references` API returns all artifacts that are linked a given manifest digest, with optional filtering. Referenced artifact requests are scoped to a repository, ensuring access rights for the repository can be used as authorization for the referenced artifacts.

Artifact references are defined in the [oci.artifact.manifest spec][oci.artifact.manifest-spec] through the [`[manifests]`][oci.artifact.manifest-spec-manifests] collection.

## Request All Artifact References

The references api is defined as an extension to the [distribution-spec][oci-distribution-spec] using the `/v2/_ext/oci-artifacts/v1/` namespace. This spec defines the behavior of the `v1` version. Clients MUST account for version checking as future major versions MAY NOT be compatible. Future Minor versions MUST be additive.

The `/references` API MUST provide for paging. The default page size SHOULD be set to 10.

```rest
GET /v2/_ext/oci-artifacts/v1/{repository}/manifests/{digest}/references?n=10
```

**expanded example:**

```rest
GET /v2/_ext/oci-artifacts/v1/net-monitor/manifests/sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b/references?n=10
```

The `/references` API MAY provide for filtering of artifactTypes. Artifact clients MUST account for [distribution-spec][oci-distribution-spec] implementations that MAY NOT support filtering. Artifact clients MUST revert to client side filtering to determine which artifactTypes they will process.

### Request Artifacts of a specific media type

**template:**
```rest
GET /v2/_ext/oci-artifacts/v1/{repository}/manifests/{digest}/references?n=10&artifactType={artifactType}
```

**expanded example:**

```rest
GET /v2/_ext/oci-artifacts/v1/net-monitor/manifests/sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b/references?n=10&artifactType=application/vnd.oci.notary.v2
```

### Artifact References API results

As not all [distribution-spec][oci-distribution-spec] implementations MAY choose to implement `artifactType` filtering, the results MAY return different artifactTypes. Other artifacts types including Notary v2 signatures, MAY return multiple signatures of the same `artifactType`. To avoid an artifact client from having to retrieve each manifest, just to determine if it's the specific artifact needed to continue processing, the `/references` API will return a collection of manifests. By providing manifests, as opposed to manifestg descriptors, a specific artifact client can find the relevant properties they need to determine which artifact to retrieve.

This paged result MUST return the following elements:

- `digest`: The digest used to retrieve the referenced manifest
- `manifest`: The contents of `oci.artifact.manifest`. By providing the manifest, consumers can choose which elements they require to determine which manifest of the paged result they must fetch.
- `@nextLink`: Used for paged results

As an example, Notary v2 manifests use annotations to determine which Notary v2 signature they should retrieve: `"org.cncf.notary.v2.signature.subject": "wabbit-networks.io"`

**example result of artifacts that reference the `net-monitor` image:**
```json
{
  "references": [
    {
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
      "manifest": {
        "schemaVersion": 1,
        "mediaType": "application/vnd.oci.artifact.manifest.v1+json",
        "artifactType": "application/vnd.cncf.notary.v2",
        "blobs": [
          {
            "mediaType": "application/tar",
            "digest": "sha256:9834876dcfb05cb167a5c24953eba58c4ac89b1adf57f28f2f9d09af107ee8f0",
            "size": 32654
          }
        ],
        "manifests": [
          {
            "mediaType": "application/vnd.oci.image.manifest.v1+json",
            "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
            "size": 16724
          }
        ],
        "annotations": {
          "org.cncf.notary.v2.signature.subject": "wabbit-networks.io"
        }
      }
    },
    {
      "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
      "manifest": {
        "schemaVersion": 1,
        "mediaType": "application/vnd.oci.artifact.manifest.v1",
        "artifactType": "application/vnd.example.sbom.v0"
      },
      "blobs": [
        {
          "mediaType": "application/tar",
          "digest": "sha256:9834876dcfb05cb167a5c24953eba58c4ac89b1adf57f28f2f9d09af107ee8f0",
          "size": 32654
        }
      ],
      "manifests": [
        {
          "mediaType": "application/vnd.oci.image.manifest.v1+json",
          "digest": "sha256:3c3a4604a545cdc127456d94e421cd355bca5b528f4a9c1905b15da2eb4a4c6b",
          "size": 16724
        }
      ],
      "annotations": {
        "example.sbom.author": "wabbit-networks.io"
      }
    }
  ],
  "@nextLink": "{opaqueUrl}"
}
```

[oci.artifact.manifest-spec]:           ./artifact-manifest-spec.md
[oci.artifact.manifest-spec-manifests]: ./artifact-manifest-spec.md#oci-artifact-manifest-properties
[oci-distribution-spec]:                https://github.com/opencontainers/distribution-spec
