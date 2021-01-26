# Introduction

This document lists all resource attributes that could be used when invoking the [Documaster API](endpoints.md). This document will help you get acquainted with the supported values for each resource in the following fields:
- `data`
- `expand`
- `expandPageSize`
- `attributes`
- `sort`
 
You can jump right to the section you are interested in:
- [Model diagram](#model-diagram)
- [Data](#data)
- [Expand](#expand)
- [Expand Page Size](#expand-page-size)
- [Attributes](#attributes)
- [Sort](#sort)
- [Model](#model)
  - [Classification](#classification)
  - [Tag](#tag)
  - [Section](#section)
  - [Entry](#entry)
  - [Party](#party)
  - [Comment](#comment)
  - [Document](#document)
  - [DocumentVersion](#document-version)
  - [ExternalId](#external-id)
  - [AccessGroup](#access-group)
  - [Model Constraints](#model-constraints)
  - [Highlights](#highlights)
  - [Effective permissions](#effective-permissions)
  - [Explicit permissions](#explicit-permissions)
    - [Explicit permissions use cases](#explicit-permissions-use-cases)
  - [Service permissions](#service-permissions)

(!) Note that any users of the Documaster API must disregard the presence of unknown attributes, (nested) resources, or parameters in any of the endpoints or resources to allow for backwards-compatible expansions of the API.

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

# "Expand Page Size"

The `expandPageSize` JSON field appears in many of the requests and resources described in the [Endpoints specification](endpoints.md).

The `expandPageSize` field allows users to specify the count of the items returned for each related resource (`documents`).

Both directly related resources (`documents`) and indirectly related resources (`documents.documentVersions`) can be specified.

The model sections below contain a list of supported `expandPageSize` values for each resource.

# "Attributes"

The `attributes` JSON field appears in many of the requests and responses described in the [Endpoints specification](endpoints.md).

The `attributes` field allows users to specify the exact attributes to be returned for the requested resource and any of its related resources listed in the `expand` field. For example, users can request that the `title` of related `tags` be returned in an Entry request as long as `tags` is specified in the `expand` field.

`attributes` of both directly and indirectly related resources can be specified as long as they appear in the `expand` field.

When `attributes` is not specified, responses will include all single-valued attributes of the requested resource.

When `attributes` is specified, the `id`, `revision`, and `effectivePermissions` attributes will always be returned for all requested resources regardless of the actual list of requested attributes.

The model sections below list all supported `attributes` values for each resource.

# "Sort"

The `sort` JSON field appears in the search-related requests and responses described in the [Endpoints specification](endpoints.md).

The `sort` field allows users to specify `attributes` according to which results will be sorted.

The `sort` field cannot contain `attributes` of related entities.

The model sections below list all attributes supported as `sort` values for each resource.

# Model

This section lists all resources, their attributes and their supported values for the `data`, `attributes`, `sort`, `expand`, and `expandPageSize` fields.

The `data` sections describe the attributes that are:
- accepted in requests
- accepted in the `attributes` field
- accepted in the `sort` field
- returned in responses

The table columns are as follows:
- `Attribute` - the attribute name
- `Sort` - whether the attribute can be sorted on. `X` means yes
- `Data type` - the data type of the attribute
- `Comment` - a description of the attribute together with any related information

The `expand` sections describe the supported values for the `expand` field in requests.

The `expandPageSize` can be set to any value that is set in the `expand` list of the particular request.

## Classification

### Data

The Classification resource is identified by `classification` in URLs.

A Classification is identified by the following attributes:

| Attribute            | Sort | Data type            | Comment                                                             |
|----------------------|------|----------------------|---------------------------------------------------------------------|
| id                   | X    | string               | (Globally) Unique identifier of the resource                        |
| revision             | X    | integer              | Revision of the resource used for concurrent modification detection |
| createdDate          | X    | timestamp            | Timestamp at which the resource was created                         |
| createdBy            | X    | string               | User name of the user who created the resource                      |
| createdByUserId      | X    | string               | User ID of the user who created the resource                        |
| updatedDate          | X    | timestamp            | Timestamp at which the resource was updated                         |
| updatedBy            | X    | string               | User name of the user who last updated the resource                 |
| updatedByUserId      | X    | string               | User ID of the user who updated the resource                        |
| title                | X    | string               | Title of the classification                                         |
| description          | X    | string               | Description of the Classification                                   |
| isSystem             | X    | boolean              | Defaults to false. Flags the Classification as System (see below).  |
| highlights           |      | highlight snippets   | Read-only [Highlight snippets](#highlights) per resource attribute  |
| explicitPermissions  |      | explicit permissions | Write-only [Explicit permissions](#explicit-permissions)            |
| effectivePermissions |      | string array         | Read-only [Effective permissions](#effective-permissions)           |
| classification       |      | array of resources   | Related Tags                                                        |
| sections             |      | array of resources   | Related Sections                                                    |
| externalIds          |      | array of resources   | Related ExternalIds                                                 |

A system Classification is only different from regular Classifications in that it will be hidden from users in the Documaster UI. Integrations are encouraged to set `isSystem` to true on Classifications that should not be visible to any users via the Documaster UI.

### Expand

A Classification accepts the following `expand` values:
- tags

## Tag

### Data

The Tag resource is identified by `tag` in URLs.

A Tag is identified by the following attributes:

| Attribute            | Sort | Data type          | Comment                                                             |
|----------------------|------|--------------------|---------------------------------------------------------------------|
| id                   | X    | string             | (Globally) Unique identifier of the resource                        |
| revision             | X    | integer            | Revision of the resource used for concurrent modification detection |
| createdDate          | X    | timestamp          | Timestamp at which the resource was created                         |
| createdBy            | X    | string             | User name of the user who created the resource                      |
| createdByUserId      | X    | string             | User ID of the user who created the resource                        |
| updatedDate          | X    | timestamp          | Timestamp at which the resource was updated                         |
| updatedBy            | X    | string             | User name of the user who last updated the resource                 |
| updatedByUserId      | X    | string             | User ID of the user who updated the resource                        |
| title                | X    | string             | Title of the Tag                                                    |
| description          | X    | string             | Description of the Tag                                              |
| highlights           |      | highlight snippets | Read-only [Highlight snippets](#highlights) per resource attribute  |
| effectivePermissions |      | string array       | Read-only [Effective permissions](#effective-permissions)           |
| classification       |      | resource           | Parent Classification                                               |
| externalIds          |      | array of resources | Related ExternalIds                                                 |

### Expand

A Tag accepts the following `expand` values:
- classification

## Section

### Data

The Section resource is identified by `section` in URLs.

A Section is identified by the following attributes:

| Attribute            | Sort | Data type            | Comment                                                             |
|----------------------|------|----------------------|---------------------------------------------------------------------|
| id                   | X    | string               | (Globally) Unique identifier of the resource                        |
| revision             | X    | integer              | Revision of the resource used for concurrent modification detection |
| createdDate          | X    | timestamp            | Timestamp at which the resource was created                         |
| createdBy            | X    | string               | User name of the user who created the resource                      |
| createdByUserId      | X    | string               | User ID of the user who created the resource                        |
| updatedDate          | X    | timestamp            | Timestamp at which the resource was updated                         |
| updatedBy            | X    | string               | User name of the user who last updated the resource                 |
| updatedByUserId      | X    | string               | User ID of the user who updated the resource                        |
| title                | X    | string               | Title of the Tag                                                    |
| description          | X    | string               | Description of the Section                                          |
| highlights           |      | highlight snippets   | Read-only [Highlight snippets](#highlights) per resource attribute  |
| explicitPermissions  |      | explicit permissions | Write-only [Explicit permissions](#explicit-permissions)            |
| effectivePermissions |      | string array         | Read-only [Effective permissions](#effective-permissions)           |
| classifications      |      | array of resources   | Related Classifications                                             |
| entries              |      | array of resources   | Related Entries                                                     |
| externalIds          |      | array of resources   | Related ExternalIds                                                 |

### Expand

A Section accepts the following `expand` values:
- classifications
- entries
- externalIds

## Entry

### Data

The Entry resource is identified by `entry` in URLs.

An Entry is identified by the following attributes:

| Attribute            | Sort | Data type          | Comment                                                             |
|----------------------|------|--------------------|---------------------------------------------------------------------|
| id                   | X    | string             | (Globally) Unique identifier of the resource                        |
| revision             | X    | integer            | Revision of the resource used for concurrent modification detection |
| createdDate          | X    | timestamp          | Timestamp at which the resource was created                         |
| createdBy            | X    | string             | User name of the user who created the resource                      |
| createdByUserId      | X    | string             | User ID of the user who created the resource                        |
| updatedDate          | X    | timestamp          | Timestamp at which the resource was updated                         |
| updatedBy            | X    | string             | User name of the user who last updated the resource                 |
| updatedByUserId      | X    | string             | User ID of the user who updated the resource                        |
| title                | X    | string             | Title of the Entry                                                  |
| description          | X    | string             | Description of the Entry                                            |
| entryDate            | X    | date               | Date of the Entry                                                   |
| highlights           |      | highlight snippets | Read-only [Highlight snippets](#highlights) per resource attribute  |
| effectivePermissions |      | string array       | Read-only [Effective permissions](#effective-permissions)           |
| section              |      | resource           | Parent Section                                                      |
| tags                 |      | array of resources | Related Tags                                                        |
| documents            |      | array of resources | Related Documents                                                   |
| externalIds          |      | array of resources | Related ExternalIds                                                 |
| parties              |      | array of resources | Related Parties                                                     |
| comments             |      | array of resources | Related Comments                                                    |

### Expand

An Entry accepts the following `expand` values:
- section
- classifications
- tags
- documents
- externalIds
- parties
- comments

## Party

### Data

The Party resource is identified by `party` in URLs.

A Party is identified by the following attributes:

| Attribute            | Sort | Data type          | Comment                                                             |
|----------------------|------|--------------------|---------------------------------------------------------------------|
| id                   | X    | string             | (Globally) Unique identifier of the resource                        |
| revision             | X    | integer            | Revision of the resource used for concurrent modification detection |
| createdDate          | X    | timestamp          | Timestamp at which the resource was created                         |
| createdBy            | X    | string             | User name of the user who created the resource                      |
| createdByUserId      | X    | string             | User ID of the user who created the resource                        |
| updatedDate          | X    | timestamp          | Timestamp at which the resource was updated                         |
| updatedBy            | X    | string             | User name of the user who last updated the resource                 |
| updatedByUserId      | X    | string             | User ID of the user who updated the resource                        |
| name                 | X    | string             | Name of the party                                                   |
| personalNumber       | X    | string             | Personal number of the party                                        |
| organizationNumber   | X    | string             | Organization number of the party                                    |
| tempPersonalNumber   | X    | string             | Temporary personal number of the party                              |
| mailingAddress       | X    | string             | Mailing address of the party                                        |
| postalCode           | X    | string             | Postal code of the party                                            |
| city                 | X    | string             | City of the party                                                   |
| country              | X    | string             | Country of the party                                                |
| emailAddress         | X    | string             | E-mail address of the party                                         |
| phoneNumber          | X    | string             | Phone number of the party                                           |
| highlights           |      | highlight snippets | Read-only [Highlight snippets](#highlights) per resource attribute  |
| effectivePermissions |      | string array       | Read-only [Effective permissions](#effective-permissions)           |
| entry                |      | resource           | Parent Entry                                                        |

### Expand

A Party accepts the following `expand` values:
- entry

## Comment

### Data

The Comment resource is identified by `comment` in URLs.

A Comment is identified by the following attributes:

| Attribute            | Sort | Data type          | Comment                                                             |
|----------------------|------|--------------------|---------------------------------------------------------------------|
| id                   | X    | string             | (Globally) Unique identifier of the resource                        |
| revision             | X    | integer            | Revision of the resource used for concurrent modification detection |
| createdDate          | X    | timestamp          | Timestamp at which the resource was created                         |
| createdBy            | X    | string             | User name of the user who created the resource                      |
| createdByUserId      | X    | string             | User ID of the user who created the resource                        |
| updatedDate          | X    | timestamp          | Timestamp at which the resource was updated                         |
| updatedBy            | X    | string             | User name of the user who last updated the resource                 |
| updatedByUserId      | X    | string             | User ID of the user who updated the resource                        |
| text                 | X    | string             | Comment text                                                        |
| entry                |      | resource           | Parent Entry                                                        |

### Expand

A Comment accepts the following `expand` values:
- entry

## Document

### Data

The Document resource is identified by `document` in URLs.

A Document is identified by the following attributes:

| Attribute            | Sort | Data type          | Comment                                                                                                                          |
|----------------------|------|--------------------|----------------------------------------------------------------------------------------------------------------------------------|
| id                   | X    | string             | (Globally) Unique identifier of the resource                                                                                     |
| revision             | X    | integer            | Revision of the resource used for concurrent modification detection                                                              |
| createdDate          | X    | timestamp          | Timestamp at which the resource was created                                                                                      |
| createdBy            | X    | string             | User name of the user who created the resource                                                                                   |
| createdByUserId      | X    | string             | User ID of the user who created the resource                                                                                     |
| updatedDate          | X    | timestamp          | Timestamp at which the resource was updated                                                                                      |
| updatedBy            | X    | string             | User name of the user who last updated the resource                                                                              |
| updatedByUserId      | X    | string             | User ID of the user who updated the resource                                                                                     |
| title                | X    | string             | Title of the Document. Recommendation: set the title to the document version filename without the extension in your integration. |
| highlights           |      | highlight snippets | Read-only [Highlight snippets](#highlights) per resource attribute                                                               |
| effectivePermissions |      | string array       | Read-only [Effective permissions](#effective-permissions)                                                                        |
| entry                |      | resource           | Parent Entry                                                                                                                     |
| externalIds          |      | resource           | Related ExternalIds                                                                                                              |
| documentVersions     |      | resource           | Related Document Versions                                                                                                        |

### Expand

A Document accepts the following `expand` values:
- entry
- externalIds
- documentVersions

## Document Version

### Data

The Document Version resource is identified by `document-version` in URLs.

A DocumentVersion is identified by the following attributes:

| Attribute            | Sort | Data type          | Comment                                                             |
|----------------------|------|--------------------|---------------------------------------------------------------------|
| id                   | X    | string             | (Globally) Unique identifier of the resource                        |
| revision             | X    | integer            | Revision of the resource used for concurrent modification detection |
| createdDate          | X    | timestamp          | Timestamp at which the resource was created                         |
| createdBy            | X    | string             | User name of the user who created the resource                      |
| createdByUserId      | X    | string             | User ID of the user who created the resource                        |
| updatedDate          | X    | timestamp          | Timestamp at which the resource was updated                         |
| updatedBy            | X    | string             | User name of the user who last updated the resource                 |
| updatedByUserId      | X    | string             | User ID of the user who updated the resource                        |
| fileName             |      | string             | File name of the document version                                   |
| checksum             |      | string             | Checksum of the document version                                    |
| checksumAlgorithm    |      | string             | Checksum algorithm of the document version                          |
| fileSize             |      | integer            | File size of the document version                                   |
| contentType          |      | string             | Content type of the document version                                |
| text                 |      | string             | Extracted text from the document version, if any                    |
| versionNumber        | X    | integer            | Version number of the document version                              |
| highlights           |      | highlight snippets | Read-only [Highlight snippets](#highlights) per resource attribute  |
| effectivePermissions |      | string array       | Read-only [Effective permissions](#effective-permissions)           |
| document             |      | resource           | Parent Document                                                     |
| electronicDocument   |      | resource           | Related ElectronicDocuemnt as uploaded by the upload endpoint       |
| externalIds          |      | resource           | Related ExternalIds                                                 |

### Expand

A DocumentVersion accepts the following `expand` values:
- document (in order to be able to request more fields than just the the parent Document ID)
- externalIds

## External ID

### Data

The External ID resource is identified by `external-id` in URLs.

An ExternalID is identified by the following attributes:

| Attribute            | Sort | Data type          | Comment                                                             |
|----------------------|------|--------------------|---------------------------------------------------------------------|
| id                   | X    | string             | (Globally) Unique identifier of the resource                        |
| revision             | X    | integer            | Revision of the resource used for concurrent modification detection |
| createdDate          | X    | timestamp          | Timestamp at which the resource was created                         |
| createdBy            | X    | string             | User name of the user who created the resource                      |
| createdByUserId      | X    | string             | User ID of the user who created the resource                        |
| updatedDate          | X    | timestamp          | Timestamp at which the resource was updated                         |
| updatedBy            | X    | string             | User name of the user who last updated the resource                 |
| updatedByUserId      | X    | string             | User ID of the user who updated the resource                        |
| externalId           | X    | string             | External ID                                                         |
| highlights           |      | highlight snippets | Read-only [Highlight snippets](#highlights) per resource attribute  |
| effectivePermissions |      | string array       | Read-only [Effective permissions](#effective-permissions)           |
| classification       |      | resource           | Parent Classification, if any                                       |
| section              |      | resource           | Parent Section, if any                                              |
| tag                  |      | resource           | Parent Tag, if any                                                  |
| entry                |      | resource           | Parent Entry, if any                                                |
| document             |      | resource           | Parent Document, if any                                             |
| documentVersion      |      | resource           | Parent Document Version, if any                                     |

### Expand

A DocumentVersion accepts the following `expand` values:
- entry
- document
- documentVersion
- externalIds

## Access Group

### Data

The Access Group resource is identified by `access-group` in URLs.

An AccessGroup is identified by the following attributes:

| Attribute          | Data type    | Comment                                                                               |
|--------------------|--------------|---------------------------------------------------------------------------------------|
| id                 | string       | (Globally) Unique identifier of the resource                                          |
| name               | string       | Human-readable name of the access group                                               |
| description        | string       | Description of the access group                                                       |
| claim              | string       | External user claim upon which a user will be assigned as a member of an access group |
| globalPermissions  | string array | [Global permissions](#explicit-permissions)                                           |
| servicePermissions | string array | [Service permissions](#service-permissions)                                           |

Note that `expand`, `expandPageSize`, `attribute`, and `sort` cannot be used when looking up access groups.

## Model constraints

There are certain model constraints in play for created* and updated* fields.

### Created fields

The following constraints apply to the `created*` fields:
* `createdBy` and `createdByUserId` must be specified together. Specifying only one of the two will result in a Bad request.
* if `createdDate` is not specified, Documaster will automatically fall back to the current date
* if `createdBy` and `createdByUserId` are not specified, Documaster will automatically fall back to the current user
* all `created*` fields require special permissions to be set that are not usually granted to regular users. Ask us for more information.

### Updated fields

The following constraints apply to the `updated*` fields:
* `updatedBy` and `updatedByUserId` must be specified together. Specifying only one of the two will result in a Bad request.
* if `updatedDate` is not specified, Documaster will automatically fall back to the current date
* if `updatedBy` and `updatedByUserId` are not specified, Documaster will automatically fall back to the current user
* all `updated*` fields require special permissions to be set that are not usually granted to regular users. Ask us for more information.

## Highlights

`highlights` are a list of text snippets that represent one or more search hits per attribute when performing free-text searches. `highlights` will only be returned in Search endpoint responses and only if there are search hits in the text-based attributes of the corresponding resource.

`highlights` have the following format when returned as part of a resource:
```json
{
    ... [other resource attributes]

    "highlights": {
      "attribute": [
          "my free-text |=hlstart|search|=hlstop=| hit",
          "my other free-text |=hlstart|search|=hlstop=| hit in the |=hlstart|search|=hlstop=| response"
      ]
    }

    ... [other resource attributes]
}
```

The following rules apply to highlights:

* `attribute` may be the name of any `attribute` of the resource in which the `highlights` appear. For example `text`;
* at most 10 highlight snippets will be returned per `attribute`;
* the `|=hlstart=|` opening "tag" and `|=hlstop=|` closing "tag" identify a highlighted word (or sequence of words) in a highlight snippet;
* each highlight snippet will contain one or more `|=hlstart=|` opening "tags" and `|=hlstop=|` closing "tags" depending in the search results;
* `highlights` for an `attribute` will be returned (if any) even if the the `attribute` itself is excluded from the response by means of specifying the `attributes` request field. It is especially useful to exclude the `text` attribute of document versions in Search requests and only rely on the `highlights` considering that the a document may contain large amounts of context.

Note that `highlights` cannot be requested via the `attributes` request field.

## Effective permissions

The effective permissions attribute is found on most of the resources. The values of that attribute determine what kind of permissions the current user has for the current resource. The `effectivePermissions` attribute will be returned in responses, but cannot be specified as part of create or update requests.

`effectivePermissions` have the following format when returned as part of a resource:
```json
{
    "effectivePermissions": [
        "Update",
        "Delete",
        "ReadChildren",
        "CreateChildren",
        "UpdateSystemFields",
        "ModifyPermissions"
    ]
}
```

The following `effectivePermissions` may be returned per resource:

- `Update`
  - indicates that the user has permissions to update the regular attributes of the current resource such as `title`, `description`, etc. See `UpdateSystemFields` for more information
  - indicates that the user has permissions to link related resources to the current resource as part of move or create operations such as:
    - linking classifications to sections
    - linking tags to entries
    - creating or moving entries in/between sections
    - creating or moving document versions in/between documents
    - etc.
- `Delete`
  - indicates that the user has permissions to delete the current resource and its children
- `ReadChildren`
  - indicates that the user has permissions to retrieve children of the current resource and their children
- `CreateChildren`
  - indicates that the user has permissions to create children in the current resource such as:
    - entries in sections
    - tags in classifications
    - parties in entries
    - document in entries
    - etc.
  - note that to create a child resource in a parent resource, the user must also have the `Update` effective permission on the parent resource
- `UpdateSystemFields`
  - indicates that the user has permissions to update the system attributes of the current resource or the system attributes of any of its children such as `created*` and `updated*` attributes
- `ModifyPermissions`
  - indicates that the user has permissions to update the [Explicit permissions](#explicit-permissions) of a resource or any of its children

Note that new effective permissions may be added in the future.

Note that `effectivePermissions` cannot be requested via the `attributes` request field.

## Explicit permissions

Explicit permissions are found on `Classification` and `Section` in the form of the `explicitPermissions` attribute and on `AccessGroup` in the form of the `globalPermissions` attribute. The values of that attribute determine the permissions assigned to a resource based on which [Effective permissions](#effective-permissions) are then calculated.

For Sections and Classifications the `explicitPermission` attribute can be specified in create and update requests, but will not be returned in responses.

For AccessGroups, the `globalPermissions` attribute can be specified in create and update requests and will be returned in responses, if not empty.

`explicitPermission`/`globalPermissions` accept the following format when used in create and update requests :
```json
{
    "explicitPermission": {
        "group identifier 1": [
            "ReadObject",
            "ReadChildren",
            "CreateChildren",
            "UpdateChildren",
            "DeleteChildren",
            "ModifyChildrenPermissions",
            "UpdateChildrenSystemFields"
        ],
        "group identifier 2": [
            ...
        ],
        ...
    }
}
```

The following `explicitPermissions`/`globalPermissions` can be specified for `Classification`, `Section`, and `AccessGroup`:

- `ReadObject`
  - grants permissions to members of the specified access group to read the current resource
  - the permission is granted automatically to every newly-created access group
- `ReadChildren`
  - grants permissions to members of the specified access group to read the children of the current resource (and the children of their children, etc.)
  - used together, `ReadObject` and `ReadChildren` grant access to the user of the specified group access to all Classifications and their Tags or to all Sections and their children (Entries, Documents, etc.) for example
- `CreateChildren`
  - grants permissions to members of the specified access group to create children of the current resource
- `UpdateChildren`
  - grants permissions to members of the specified access group to update regular attributes of the children of the current resource such as `title`, `description`, etc.
- `DeleteChildren`
  - grants permissions to members of the specified access group to delete children of the current resource
- `ModifyChildrenPermissions`
  - grants permissions to members of the specified access group to modify the permissions assigned to children of the current resource
  - not currently in use
- `UpdateChildrenSystemFields`
  - grants permissions to members of the specified access group to update system attributes of the children of the current resource such as `created*` and `updated*` attributes

Note that new explicit permissions may be added in the future.

### Explicit permissions use cases

#### Users with global read access to the system

To grant users access to read everything in the archive create or update their access group with the following explicit/global permissions:

- `ReadObject`
- `ReadChildren`

#### Global administrators of the system

To grant users access to read and edit everything in the archive create or update their access group with the following explicit/global permissions:

- `ReadObject`
- `ReadChildren`
- `CreateChildren`
- `UpdateChildren`
- `UpdateChildrenSystemFields`

You could optionally grant `DeleteChildren`, but Documaster recommends that only a small number of people have rights to delete data in the archive.

#### Global administrators of the system with rights to grant or revoke permissions to resources

To grant users access to read and edit everything in the archive, including permissions on resources, create or update their access group with the following explicit/global permissions:

- `ReadObject`
- `ReadChildren`
- `CreateChildren`
- `UpdateChildren`
- `UpdateChildrenSystemFields`
- `ModifyChildrenPermissions`

You could optionally grant `DeleteChildren`, but Documaster recommends that only a small number of people have rights to delete data in the archive.

#### Users with access only to a particular Section in the archive

To grant users access to read and edit everything only in a single Section in the archive, create or update their access group with the following explicit/global permissions:

- `ReadObject`

Additionally, update the Section you would like to grant them access to like so:

```json
{
    "explicitPermissions": {
        "user access group ID": [
            "ReadObject", "ReadChildren", "CreateChildren", "UpdateChildren", "UpdateChildrenSystemFields"
        ]
    }
}
```

You could optionally grant `DeleteChildren` on the Sections, but Documaster recommends that only a small number of people have rights to delete data in the archive.

Finally, update all Classifications linked to the aforementioned Section like so:

```json
{
    "explicitPermissions": {
        "user access group ID": [
            "ReadObject", "ReadChildren"
        ]
    }
}
```

You could optionally grant `CreateChildren` and `UpdateChildren` on the Classifications if you would like them to be able to create or edit existing Tags.

You could optionally grant `DeleteChildren` on the Classifications, but Documaster recommends that only a small number of people have rights to delete data in the archive.

## Service permissions

The service permissions attribute is found only on `AccessGroup`. The values of that attribute determine the types of actions members of an access group can perform in Documaster. The `servicePermissions` attribute can be specified in create and update requests and will be returned in responses.

`servicePermissions` accept the following format when used in create and update requests :
```json
{
    "servicePermissions": [
        "UploadDocuments",
        "ManageAccessGroups"
    ]
}
```

The following `servicePermissions` can be specified for `AccessGroup`:

- `UploadDocuments`
  - grants permissions to members of the specified access group to upload documents in the system
- `ManageAccessGroups`
  - grants permissions to members of the specified access group to manage the access groups in the system (create, update, and delete)

Note that new service permissions may be added in the future.
