# OCI Artifact Manifest Spec

The OCI artifact manifest generalizes the use of [OCI image manifest][oci-image-manifest-spec]. It provides a means to define a wide range of artifacts, including a chain of related artifacts enabling SBoMs, Signatures and metadata. The decision to use `oci.artifact.manifest`, [oci.image.manifest][oci-image-manifest-spec] or [oci.image.index][oci-image-index] is up to the authors of specific artifact types.

For usage and scenarios, see [artifact-manifest.md](./artifact-manifest.md)

## Example OCI Artifact Manifests

- [`net-monitor:v1` container image](./artifact-manifest/net-monitor-image.json)
- [`net-monitor:v1` notary v2 signature](./artifact-manifest/net-monitor-image-signature.json)
- [`net-monitor:v1` sbom](./artifact-manifest/net-monitor-image-sbom.json)
- [`net-monitor:v1` nydus image](./artifact-manifest/net-monitor-nydus-image.json)

## OCI Artifact Manifest Properties

An artifact manifest provides configuration, a collection of blobs and an optional collection of manifests to other artifacts.

- **`schemaVersion`** *int*

  This REQUIRED property specifies the artifact manifest schema version.
  For this version of the specification, this MUST be `1`. The value of this field WILL change as the manifest schema evolves. Minor version changes to the `oci.artifact.manifest` spec MUST be additive, while major version changes MAY be breaking. Artifact clients MUST implement version checking to allow for future, yet unknown changes. Artifact clients MUST ignore additive properties to minor versions. Artifact clients MAY support major changes, with no guarantee major changes MAY impose breaking changing behaviors. Artifact authors MAY support new and older schemaVersions to provide the best user experience.

- **`mediaType`** *string*

  This field contains the `mediaType` of this document, differentiating from [image-manifest][oci-image-manifest-spec] and [oci-image-index]. The mediaType for this manifest type MUST be `application/vnd.oci.artifact.manifest.v1+json`, where the version WILL change to reflect newer versions. Artifact authors SHOULD support multiple `mediaType` versions to provide the best user experience for their artifact type.

- **`artifactType`** *string*

  A unique value, [defining the type of unique artifact type][artifact-type], as registered with iana.org. See [registering unique types.][registering-iana]. OCI Artifacts that used `oci.image.manifest` used the `manifest.config.mediaType` to differentiate the type of artifact, differentiating from a container image. Artifact authors that implement `oci.artifact.manifest` use `artifactType` to differentiate the type of artifact, freeing up the `config.mediaType` to be OPTIONAL, or shared config schemas between multiple artifact types.

- **`blobs`** *array of objects*

    A collection of 0 or more blobs. The blobs array is analogous to [oci.image.manifest layers][oci-image-manifest-spec-layers], however unlike [image-manifest][oci-image-manifest-spec], the ordering of blobs is specific to the artifact type. Some artifacts may choose an overlay of files, while other artifact types may store collections of files in different locations. To reflect the artifact specific implementations, the `oci.image.manifest.layers` collection is renamed to `blobs`.

    The [image-config][oci-config] would be considered another blob with a unique `mediaType`. See the [net-monitor-image-artifact.json](./artifact-manifest/net-monitor-image-artifact.json) for an example.

    Each item in the array MUST be a [descriptor][descriptor].

    The max number of blobs is not defined, but MAY be limited by [distribution-spec][oci-distribution-spec] implementations.

    An encountered `mediaType` that is unknown to the implementation MUST be ignored.

- **`manifests`** *array of objects*

   An OPTIONAL collection of referenced manifests, which represent linked artifacts. When used, the artifact defined within this manifest is said to be dependent upon the referenced `manifests`.

   As manifests within the `[manifests]` array are deleted, this manifest MAY be deleted. If all `[manifests]` are deleted, and this manifest has no tag reference, the registry MAY delete, based on registry specific use cases. [Distribution spec][oci-distribution-spec] implementations that implement de-duplication, MAY implement ref counting.

   Artifacts authors that implement the `oci.artifact.manifest` MAY support tagged references to the artifact. Artifacts that reference other artifacts through the use of `[manifests]` are said to be dependent upon the referenced artifact and not typically directly referenced through tags. When an artifact has both a `[manifests]` entry, and a tag, the [distribution-spec][oci-distribution-spec] implementation MAY maintain the manifest as the tag reference is expected to be individually referenced. As the manifest is dereferenced from a tag, and all `[manifests]` entries are removed, the manifest SHOULD be deleted.

    Each item in the array MUST be a [descriptor][descriptor] representing a manifest.

- **`annotations`** *string-string map*

    This OPTIONAL property contains arbitrary metadata for the image manifest.
    This OPTIONAL property MUST use the [annotation rules](annotations.md#rules).

    See [Pre-Defined Annotation Keys](annotations.md#pre-defined-annotation-keys).

## Annotations

OCI Artifact Manifest includes several annotations that have been generalized from the image-spec annotations.

- **`annotations`** *string-string map*

    This OPTIONAL property contains arbitrary metadata for the image manifest.
    This OPTIONAL property MUST use the [annotation rules](annotations.md#rules).

    See [Pre-Defined Annotation Keys](annotations.md#pre-defined-annotation-keys).

### Pre-Defined Annotation Keys

This specification defines the following annotation keys, intended for but not limited to  Artifact Manifest authors:

- **org.oci.artifact.created** date and time on which the artifact was built (string, date-time as defined by [RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6)).
- **org.oci.artifact.authors** contact details of the people or organization responsible for the artifact (semi-colon delineated string)
- **org.oci.artifact.url** URL to find more information on the artifact (string)
- **org.oci.artifact.documentation** URL to get documentation on the artifact (string)
- **org.oci.artifact.source** URL to get source code for building the artifact (string)
- **org.oci.artifact.version** version of the packaged software
  - The version MAY match a label or tag in the source code repository
  - version MAY be [Semantic versioning-compatible](http://semver.org/)
- **org.oci.artifact.revision** Source control revision identifier for the packaged software.
- **org.oci.artifact.vendor** Name of the distributing entity, organization or individual.
- **org.oci.artifact.title** Human-readable title of the artifact (string)
- **org.oci.artifact.description** Human-readable description of the software packaged in the artifact (string)

```json
{
  "annotations": {
    "org.oci.artifact.created": "",
    "org.oci.artifact.authors": "",
    "org.oci.artifact.url": "opencontainers.org",
    "org.oci.artifact.documentation": "opencontainers.org",
    "org.oci.artifact.source": "https://github.com/opencontainers/artifacts",
    "org.oci.artifact.version": "v1.0",
    "org.oci.artifact.revision": "v1.1.0",
    "org.oci.artifact.vendor": "Open Containers Initiative",
    "org.oci.artifact.licenses": "MIT",
    "org.oci.artifact.title": "Open Containers Artifact Manifest",
    "org.oci.artifact.description": "A schema for defining artifacts"
  }
}
```

## Q&A

- **Q**: Where is the `config` property?
  - **A**: The `oci.image.manifest.config` object is a [descriptor][descriptor] stored as a blob, alongside the other `[layers]`. Instead of making this an OPTIONAL property on `oci.artifact.manifest`, artifact types that require a config can simply identify a blob as a config object. See the [net-monitor-image-artifact.json](./artifact-manifest/net-monitor-image-artifact.json) for an example.

[oci-config]:                      https://github.com/opencontainers/image-spec/blob/master/config.md
[oci-image-manifest-spec]:         https://github.com/opencontainers/image-spec/blob/master/manifest.md
[oci-image-manifest-spec-layers]:  https://github.com/opencontainers/image-spec/blob/master/manifest.md#image-manifest-property-descriptions
[oci-image-index]:                 https://github.com/opencontainers/image-spec/blob/master/image-index.md
[oci-distribution-spec]:           https://github.com/opencontainers/distribution-spec
[media-type]:                      https://github.com/opencontainers/image-spec/blob/master/media-types.md
[artifact-type]:                   https://github.com/opencontainers/artifacts/blob/master/artifact-authors.md#defining-a-unique-artifact-type
[registering-iana]:                ./artifact-authors.md#registering-unique-types-with-iana
[descriptor]:                      https://github.com/opencontainers/image-spec/blob/master/descriptor.md
