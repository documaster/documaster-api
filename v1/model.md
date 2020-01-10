# Introduction

This document lists all resource attributes that could be used when invoking the [Documaster API](endpoints.md). This document will help you get acquainted with the supported values for each resource in the following fields:
- `data`
- `expand`
- `attributes`
- `sort`
 
You can jump right to the section you are interested in:
- [Model diagram](#model-diagram)
- [Data](#data)
- [Expand](#expand)
- [Attributes](#attributes)
- [Sort](#sort)
- [Model](#model)
  - [Classification](#classification)
  - [Tag](#tag)
  - [Section](#section)
  - [Entry](#entry)
  - [Party](#party)
  - [Document](#document)
  - [DocumentVersion](#document-version)
  - [ExternalId](#external-id)

(!) Note that any users of the Documaster API must disregard the presense of unknown attributes, (nested) resources, or parameters in any of the endpoints or resources to allow for backwards-compatible expansions of the API.

---

# Model diagram

The following diagram illustrates the resources exposed via the Documaster API.

![Model diagram](img/model.png)

# "Data"

The `data` JSON field appears in many of the requests and responses described in the [Endpoints specification](endpoints.md).

The `data` field contains all attribute-value pairs of a resource as described below. These can be specified in create, update, and lookup request and will be returned in responses.

The model sections below contain a list of supported `attributes` that users can specify for each resource or expect that they will be returned in responses.

# "Expand"

The `expand` JSON field appears in many of the requests and responses described in the [Endpoints specification](endpoints.md).

The `expand` field allows users to specify related resources to be included in the response. For example, users can request that the related `tags` and `externalIds` be returned in an `entry` request.

Both directly related resources (`tags`) and indirectly related resources (`tags.classification`) can be specified.

The model sections below contain a list of supported `expand` values for each resource.

# "Attributes"

The `attributes` JSON field appears in many of the requests and responses described in the [Endpoints specification](endpoints.md).

The `attributes` field allows users to specify the exact attributes to be returned for the requested resource and any of its related resources listed in the `expand` field. For example, users can request that the `title` of related `tags` be returned in an Entry request as long as `tags` is specified in the `expand` field.

`attributes` of both directly and indirectly related resources can be specified as long as they appear in the `expand` field.

When `attributes` is not specified, responses will include:
- all single-valued attributes of the requested resource
- the ID attribute of all related many- or one-to-one references

The model sections below list all supported `attributes` values for each resource.

# "Sort"

The `sort` JSON field appears in the search-related requests and responses described in the [Endpoints specification](endpoints.md).

The `sort` field allows users to specify `attributes` according to which results will be sorted.

The `sort` field cannot contain `attributes` of related entities.

The model sections below list all attributes supported as `sort` values for each resource.

# Model

This section lists all resources, their attributes and their supported values for the `data`, `attributes`, `sort`, and `expand` fields.

The `data` sections describe the attributes that are:
- accepted in requests
- accepted in the `attributes` field
- accepted in the `sort` field
- returned in responses

The table columns are as follows:
- `Attribute` - the attribute name
- `Response` - whether the attribute will be returned in the response by default. `X` means yes
- `Sort` - whether the attribute can be sorted on. `X` means yes
- `Data type` - the data type of the attribute
- `Comment` - a description of the attribute together with any related information

The `expand` sections describe the supported values for the `expand` field in requests.

## Classification

### Data

A Classification is identified by the following attributes:

| Attribute       | Response | Sort | Data type          | Comment                                                             |
|-----------------|----------|------|--------------------|---------------------------------------------------------------------|
| id              | X        | X    | string             | (Globally) Unique identifier of the resource                        |
| revision        | X        | X    | integer            | Revision of the resource used for concurrent modification detection |
| createdDate     | X        | X    | timestamp          | Timestamp at which the resource was created                         |
| createdBy       | X        | X    | string             | User name of the user who created the resource                      |
| createdByUserId | X        | X    | string             | User ID of the user who created the resource                        |
| title           | X        | X    | string             | Title of the classification                                         |
| description     | X        | X    | string             | Description of the Classification                                   |
| tags            |          |      | array of resources | Related Tags                                                        |

### Expand

A Classification accepts the following `expand` values:
- tags

## Tag

### Data

A Tag is identified by the following attributes:

| Attribute       | Response | Sort | Data type | Comment                                                             |
|-----------------|----------|------|-----------|---------------------------------------------------------------------|
| id              | X        | X    | string    | (Globally) Unique identifier of the resource                        |
| revision        | X        | X    | integer   | Revision of the resource used for concurrent modification detection |
| createdDate     | X        | X    | timestamp | Timestamp at which the resource was created                         |
| createdBy       | X        | X    | string    | User name of the user who created the resource                      |
| createdByUserId | X        | X    | string    | User ID of the user who created the resource                        |
| title           | X        | X    | string    | Title of the classification                                         |
| description     | X        | X    | string    | Description of the classification                                   |
| classification  | X        |      | resource  | Parent Classification. By default, includes only the `id` attribute |

### Expand

A Tag accepts the following `expand` values:
- classification (in order to be able to request more fields than just the the parent Classification ID)

## Section

### Data

A Section is identified by the following attributes:

| Attribute       | Response | Sort | Data type          | Comment                                                             |
|-----------------|----------|------|--------------------|---------------------------------------------------------------------|
| id              | X        | X    | string             | (Globally) Unique identifier of the resource                        |
| revision        | X        | X    | integer            | Revision of the resource used for concurrent modification detection |
| createdDate     | X        | X    | timestamp          | Timestamp at which the resource was created                         |
| createdBy       | X        | X    | string             | User name of the user who created the resource                      |
| createdByUserId | X        | X    | string             | User ID of the user who created the resource                        |
| title           | X        | X    | string             | Title of the classification                                         |
| description     | X        | X    | string             | Description of the Classification                                   |
| classifications |          |      | array of resources | Related Classifications                                             |

### Expand

A Section accepts the following `expand` values:
- classifications

## Entry

### Data

An Entry is identified by the following attributes:

| Attribute       | Response | Sort | Data type          | Comment                                                             |
|-----------------|----------|------|--------------------|---------------------------------------------------------------------|
| id              | X        | X    | string             | (Globally) Unique identifier of the resource                        |
| revision        | X        | X    | integer            | Revision of the resource used for concurrent modification detection |
| createdDate     | X        | X    | timestamp          | Timestamp at which the resource was created                         |
| createdBy       | X        | X    | string             | User name of the user who created the resource                      |
| createdByUserId | X        | X    | string             | User ID of the user who created the resource                        |
| title           | X        | X    | string             | Title of the classification                                         |
| description     | X        | X    | string             | Description of the Classification                                   |
| section         | X        |      | resource           | Parent Section. By default, includes only the `id` attribute        |
| classifications |          |      | array of resources | Related Classifications                                             |
| tags            |          |      | array of resources | Related Tags                                                        |
| documents       |          |      | array of resources | Related Documents                                                   |
| externalIds     |          |      | array of resources | Related ExternalIds                                                 |
| parties         |          |      | array of resources | Related Parties                                                     |

### Expand

An Entry accepts the following `expand` values:
- section (in order to be able to request more fields than just the the parent Section ID)
- classifications
- tags
- documents
- externalIds
- parties

## Party

### Data

A Party is identified by the following attributes:

| Attribute          | Response | Sort | Data type | Comment                                                             |
|--------------------|----------|------|-----------|---------------------------------------------------------------------|
| id                 | X        | X    | string    | (Globally) Unique identifier of the resource                        |
| revision           | X        | X    | integer   | Revision of the resource used for concurrent modification detection |
| createdDate        | X        | X    | timestamp | Timestamp at which the resource was created                         |
| createdBy          | X        | X    | string    | User name of the user who created the resource                      |
| createdByUserId    | X        | X    | string    | User ID of the user who created the resource                        |
| name               | X        | X    | string    | Name of the party                                                   |
| personalNumber     | X        | X    | string    | Personal number of the party                                        |
| organizationNumber | X        | X    | string    | Organization number of the party                                    |
| tempPersonalNumber | X        | X    | string    | Temporary personal number of the party                              |
| mailingAddress     | X        | X    | string    | Mailing address of the party                                        |
| postalCode         | X        | X    | string    | Postal code of the party                                            |
| city               | X        | X    | string    | City of the party                                                   |
| country            | X        | X    | string    | Country of the party                                                |
| emailAddress       | X        | X    | string    | E-mail address of the party                                         |
| phoneNumber        | X        | X    | string    | Phone number of the party                                           |
| entry              | X        |      | resource  | Parent Entry. By default, includes only the `id` attribute          |

### Expand

A Party accepts the following `expand` values:
- entry (in order to be able to request more fields than just the the parent Entry ID)

## Document

### Data

A Document is identified by the following attributes:

| Attribute        | Response | Sort | Data type | Comment                                                             |
|------------------|----------|------|-----------|---------------------------------------------------------------------|
| id               | X        | X    | string    | (Globally) Unique identifier of the resource                        |
| revision         | X        | X    | integer   | Revision of the resource used for concurrent modification detection |
| createdDate      | X        | X    | timestamp | Timestamp at which the resource was created                         |
| createdBy        | X        | X    | string    | User name of the user who created the resource                      |
| createdByUserId  | X        | X    | string    | User ID of the user who created the resource                        |
| title            | X        | X    | string    | Title of the Document                                               |
| entry            | X        |      | resource  | Parent Entry. By default, includes only the `id` attribute          |
| externalIds      |          |      | resource  | Related ExternalIds                                                 |
| documentVersions |          |      | resource  | Related Document Versions                                           |

### Expand

A Document accepts the following `expand` values:
- entry (in order to be able to request more fields than just the the parent Entry ID)
- externalIds
- documentVersions

## Document Version

### Data

A DocumentVersion is identified by the following attributes:

| Attribute          | Response | Sort | Data type | Comment                                                             |
|--------------------|----------|------|-----------|---------------------------------------------------------------------|
| id                 | X        | X    | string    | (Globally) Unique identifier of the resource                        |
| revision           | X        | X    | integer   | Revision of the resource used for concurrent modification detection |
| createdDate        | X        | X    | timestamp | Timestamp at which the resource was created                         |
| createdBy          | X        | X    | string    | User name of the user who created the resource                      |
| createdByUserId    | X        | X    | string    | User ID of the user who created the resource                        |
| fileName           | X        | X    | string    | File name of the document version                                   |
| checksum           | X        | X    | string    | Checksum of the document version                                    |
| checksumAlgorithm  | X        | X    | string    | Checksum algorithm of the document version                          |
| fileSize           | X        | X    | integer   | File size of the document version                                   |
| contentType        | X        | X    | string    | Content type of the document version                                |
| versionNumber      | X        | X    | integer   | Version number of the document version                              |
| document           | X        |      | resource  | Parent Document. By default, includes only the `id` attribute       |
| electronicDocument | X        |      | resource  | Related ElectronicDocuemnt as uploaded by the upload endpoint       |
| externalIds        |          |      | resource  | Related ExternalIds                                                 |

### Expand

A DocumentVersion accepts the following `expand` values:
- document (in order to be able to request more fields than just the the parent Document ID)
- externalIds

## External ID

### Data

An ExternalID is identified by the following attributes:

| Attribute       | Response | Sort | Data type | Comment                                                                       |
|-----------------|----------|------|-----------|-------------------------------------------------------------------------------|
| id              | X        | X    | string    | (Globally) Unique identifier of the resource                                  |
| revision        | X        | X    | integer   | Revision of the resource used for concurrent modification detection           |
| createdDate     | X        | X    | timestamp | Timestamp at which the resource was created                                   |
| createdBy       | X        | X    | string    | User name of the user who created the resource                                |
| createdByUserId | X        | X    | string    | User ID of the user who created the resource                                  |
| externalId      | X        | X    | string    | External ID                                                                   |
| entry           |          |      | resource  | Parent Entry, if any. By default, includes only the `id` attribute            |
| document        |          |      | resource  | Parent Document, if any. By default, includes only the `id` attribute         |
| documentVersion |          |      | resource  | Parent Document Version, if any. By default, includes only the `id` attribute |

### Expand

A DocumentVersion accepts the following `expand` values:
- document (in order to be able to request more fields than just the the parent Document ID)
- externalIds
