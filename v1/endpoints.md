# Introduction

This document specifies information about the Documaster API endpoints:
- [Basic concepts](#basic-concepts)
- [Resource Endpoints](#resource-endpoints)
  - [Fetch endpoint](#fetch-endpoint) - GET /{resource}
  - [Fetch by ID endpoint](#fetch-by-id-endpoint) - GET /{resource}/{id}
  - [Fetch related endpoint](#fetch-related-endpoint) - GET /{resource}/{id}/{related-resource}
  - [Create endpoint](#create-endpoint) - POST /{resource}
  - [Update endpoint](#update-endpoint) - PUT /{resource}
  - [Delete endpoint](#delete-endpoint) - DELETE /{resource}
  - [Lookup](#lookup-endpoint) - POST /{resource}/lookup
  - [Search](#search-endpoint) - POST /{resource}/search
  - [Upload](#upload-endpoint) - POST upload-temp/
  - [Download](#download-endpoint) - GET download/{id}
- [Access Group Endpoints](#access-group-endpoints)
  - [Fetch access groups endpoint](#fetch-all-access-groups-endpoint) - GET /access-group
  - [Fetch user access groups endpoint](#fetch-my-access-groups-endpoint) - GET /access-group/my
  - [Fetch access group by ID endpoint](#fetch-access-group-by-id-endpoint) - GET /access-group/{id}
  - [Create access group endpoint](#create-access-group-endpoint) - POST /access-group
  - [Update access group endpoint](#update-access-group-endpoint) - PUT /access-group
  - [Delete access group endpoint](#delete-access-group-endpoint) - DELETE /access-group
- [Error response format](#error-response-format)

Refer to the [Model specification](model.md) for more information on the `{resource}` path parameter and the contents of `data`, `expand`, `expandPageSize` and `attributes` used in this document.

Additionally, information about the [error response format](#error-response-format) can be found at the end of the document.

(!) Note that any users of the Documaster API must disregard the presence of unknown attributes, (nested) resources, or parameters in any of the endpoints to allow for backwards-compatible expansions of the API.

---

# Basic concepts

## Supported protocols

The web services support the following protocols:

- HTTPS

## Authentication and authorization

The web services require an access token (Bearer) to be specified in the HTTP headers when sending requests. A token can be obtained from the OpenID Connect Identity Provider used by the Documaster instance.

## HTTP status codes

The following table lists the HTTP status codes used by the web services.

| Status code                | Definition                                                                                                                                         | Example                                                         |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| 200 OK                     | The request was successful.                                                                                                                        | HTTP/1.1 200 OK                                                 |
| 201 Created                | The request was fulfilled and resulted in a new resource being created.                                                                            | HTTP/1.1 201 Created                                            |
| 204 No Content             | The request was successful, but there is no content to send in the response payload body.                                                          | HTTP/1.1 204 No Content                                         |
| 400 Bad Request            | The syntax of the request was invalid.                                                                                                             | HTTP/1.1 400 Bad Request                                        |
| 401 Unauthorized           | The request was denied due to an invalid or missing access token.                                                                                  | <p>HTTP/1.1 401 Unauthorized</p><p>WWW-Authenticate: Bearer</p> |
| 403 Forbidden              | The request was denied due to the access token having insufficient privileges to access the requested resource or perform the requested operation. | HTTP/1.1 403 Forbidden                                          |
| 404 Not Found              | The requested endpoint does not exist.                                                                                                             | HTTP/1.1 404 Not Found                                          |
| 409 Conflict               | Data was updated by another client after the last read by this client. The client may retry the request.                                           | HTTP/1.1 409 Conflict                                           |
| 415 Unsupported Media Type | The payload format requested by the client is not supported by the server.                                                                         | HTTP/1.1 415 Unsupported Media Type                             |
| 500 Internal Server Error  | An internal server error has occurred.                                                                                                             | HTTP/1.1 500 Internal Server Error                              |
| 503 Service Unavailable    | The service is temporarily unavailable.                                                                                                            | HTTP/1.1 503 Service Unavailable                                |
## Path

All endpoints appear under the following base path:

**{baseUrl}/api/v1/{resource}**

## Dates encoding

- Dates must be encoded as **ISO 8601** strings prior to sending them to the web services.
- Dates are returned by the web services as **ISO 8601** formatted strings.

In plain English:
- `timestamp` attributes have to be be in the following format: `2020-01-01T01:01:01.000+01:00`
- `date` attributes have to be be in the following format: `2020-01-01`

---

# Resource Endpoints

## Fetch endpoint

Retrieve the resources of the specified type, if any.

### Request

```
GET /{resource}?page=1&pageSize=10&flags=includeTotal&expand=resource1,resource2&attributes=field1,resource1.field1&sortField=title&sortOrder=desc HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
```

- `page` (optional)
  - specifies the page to fetch; 1-based
  - defaults to 1
- `pageSize` (optional)
  - specifies the page size
  - defaults to 10
  - maximum allowed value is 1000
- `flags` (optional)
  - specifies one or more flags requesting additional information
  - `includeTotal`
    - retrieves the total amount of results
    - incurs a performance cost on the Documaster instance
    - defaults to false
- `expand` (optional)
  - specifies additional resources to be included in the response
- `attributes` (optional)
  - specifies the list of attributes to be returned for each resource in the response
- `sortField` (optional)
  - specifies the field of the fetched object by which the response to be sorted. Works only in conjunction with **sortOrder**
- `sortOrder` (optional)
  - specifies the sort order in which the response to be ordered asc or desc

### Response

```
200 OK
Content-type: application/json

{
    "data": [
        {
            attribute: value,
            ...,
            "resources": {
                "self": "{resource}/{id}",
                attribute: {
                    "self": "{resource}/{id}/{related-resource}"
                    "hasMore": boolean,
                    "page": integer,
                    "pageSize": integer
                },
                ...
            }
        },
        ...,
    ],
    "hasMore": boolean,
    "page": integer,
    "pageSize": integer,
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
    "attributes": [string, ...],
}
```

- `data`
  - contains a (potentially empty) list of the resources matching the specified lookup request
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `resources`
- `resources`
  - lists additional information per returned resource
  - `self`
    - contains a link to fetch the current resource
  - `attribute` (optional)
    - denotes a collection of related entities by their attribute (tags, documents, etc.)
    - `self`
      - contains a link to fetch more of the related entities
    - `hasMore` (optional)
      - specifies whether more pages exist
      - only returned for related resources that represent a collection
    - `page` (optional)
      - the requested page
      - only appears for related resources that represent a collection
    - `pageSize` (optional)
      - the requested page size
      - only returned for related resources that represent a collection
- `hasMore`
  - specifies whether more pages exist
- `page`
  - the requested page
- `pageSize`
  - the requested page size
- `flags`
  - the requested list of flags
- `expand` (optional)
  - the requested list of additional resources
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `200 OK`
- `400 Bad request`

---

## Fetch by ID endpoint

Retrieves the resources with the specified type and ID, if any.

### Request

```
GET /{resource}/{id}?expand=resource1,resource2&attributes=field1,resource1.field1?flags=includeTotal HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
```

- `flags` (optional)
  - specifies one or more flags requesting additional information
  - `includeTotal`
    - retrieves the total amount of results
    - incurs a performance cost on the Documaster instance
    - defaults to false
- `expand` (optional)
  - specifies additional resources to be included in the response
- `attributes` (optional)
  - specifies the list of attributes to be returned for each resource in the response

See the model specification for more information on the contents of 
`expand` and `attributes`.

### Response

```
200 OK
Content-type: application/json

{
    "data": {
        attribute: value,
        ...,
        "resources": {
            "self": "{resource}/{id}",
            attribute: {
                "self": "{resource}/{id}/{related-resource}"
                "hasMore": boolean,
                "page": integer,
                "pageSize": integer
            },
            ...
        }
    },
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
    "attributes": [string, ...],
}
```

- `data`
  - contains the (potentially empty) resource with the specified ID
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `resources`
- `resources`
  - lists additional information per returned resource
  - `self`
    - contains a link to fetch the current resource
  - `attribute` (optional)
    - denotes a collection of related entities by their attribute
    - `self`
      - contains a link to fetch more of the related entities
    - `hasMore` (optional)
      - specifies whether more pages exist
      - only returned for related resources that represent a collection
    - `page` (optional)
      - the requested page
      - only returned for related resources that represent a collection
    - `pageSize` (optional)
      - the requested page size
      - only returned for related resources that represent a collection
- `flags`
  - the requested list of flags
- `expand` (optional)
  - the requested list of additional resources
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `200 OK`
- `400 Bad request`
- `403 Unauthorized`

---

## Fetch Related endpoint

Retrieves resources related to a resource with the specified type and ID, if any.

### Request

```
GET /{resource}/{id}/{related-resource}?expand=resource1,resource2&attributes=field1,resource1.field1?flags=includeTotal&page=1&pageSize=10 HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
```

- `page` (optional)
  - specifies the page to fetch; 1-based
  - defaults to 1
- `pageSize` (optional)
  - specifies the page size
  - defaults to 10
  - maximum allowed value is 1000
- `flags` (optional)
  - specifies one or more flags requesting additional information
  - `includeTotal`
    - retrieves the total amount of results
    - incurs a performance cost on the Documaster instance
    - defaults to false
- `expand` (optional)
  - specifies additional resources to be included in the response
- `attributes` (optional)
  - specifies the list of attributes to be returned for each resource in the response

See the model specification for more information on the contents of 
`expand` and `attributes`.

### Response

```
200 OK
Content-type: application/json

{
    "data": [
        {
            attribute: value,
            ...,
            "resources": {
                "self": "{resource}/{id}",
                attribute: {
                    "self": "{resource}/{id}/{related-resource}"
                    "hasMore": boolean,
                    "page": integer,
                    "pageSize": integer
                },
                ...
            }
        },
        ...,
    ],
    "hasMore": boolean,
    "page": integer,
    "pageSize": integer,
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
    "attributes": [string, ...],
}
```

- `data`
  - contains a (potentially empty) list of the resources matching the specified lookup request
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `resources`
- `resources`
  - lists additional information per returned resource
  - `self`
    - contains a link to fetch the current resource
  - `attribute` (optional)
    - denotes a collection of related entities by their attribute (tags, documents, etc.)
    - `self`
      - contains a link to fetch more of the related entities
    - `hasMore` (optional)
      - specifies whether more pages exist
      - only returned for related resources that represent a collection
    - `page` (optional)
      - the requested page
      - only appears for related resources that represent a collection
    - `pageSize` (optional)
      - the requested page size
      - only returned for related resources that represent a collection
- `hasMore`
  - specifies whether more pages exist
- `page`
  - the requested page
- `pageSize`
  - the requested page size
- `flags`
  - the requested list of flags
- `expand` (optional)
  - the requested list of additional resources
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `200 OK`
- `400 Bad request`
- `403 Unauthorized`

---

## Create endpoint

Creates a new resource.

### Request

```
POST /{resource} HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
Content-type: application/json

{
    "data": { attribute: value, ... },
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
    "expandPageSize": {"string": integer, ... },
    "attributes": [ string, ... ]
}
```

- `data`
  - contains the resource to be created
  - at most one thousand (1000) nested resources in total can be created as part of the same request
- `flags` (optional)
  - specifies one or more flags requesting additional information
  - `includeTotal`
    - retrieves the total amount of results
    - incurs a performance cost on the Documaster instance
    - defaults to false
- `expand` (optional)
  - specifies additional resources to be included in the response
- `expandPageSize` (optional)
  - specifies the page size for the additional resources in the `expand` parameter
- `attributes` (optional)
  - specifies the list of attributes to be returned for each resource in the response

### Response

```
201 Created
Content-type: application/json

{
    "data": {
        attribute: value,
        ...,
        "resources": {
            "self": "{resource}/{id}",
            attribute: {
                "self": "{resource}/{id}/{related-resource}"
                "hasMore": boolean,
                "page": integer,
                "pageSize": integer
            },
            ...
        }
    },
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
    "expandPageSize": {"string": integer, ... },
    "attributes": [ string, ... ]
}
```

- `data`
  - contains the created resource
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `resources`
- `resources`
  - lists additional information per returned resource
  - `self`
    - contains a link to fetch the current resource
  - `attribute` (optional)
    - denotes a collection of related entities by their attribute
    - `self`
      - contains a link to fetch more of the related entities
    - `hasMore` (optional)
      - specifies whether more pages exist
      - only returned for related resources that represent a collection
    - `page` (optional)
      - the requested page
      - only returned for related resources that represent a collection
    - `pageSize` (optional)
      - the requested page size
      - only returned for related resources that represent a collection
- `flags`
  - the requested list of flags
- `expand` (optional)
  - the requested list of additional resources
- `expandPageSize` (optional)
  - the requested map of page sizes for the additional resources in the `expand` parameter
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `201 Created`
- `400 Bad request`
- `403 Unauthorized`

---

## Update endpoint

Updates an existing resource.

### Request

```
PUT /{resource}/{id} HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
Content-type: application/json

{
    "data": { attribute: value, ... },
    "update": {
        attribute: [
            { "add": { "id": value } }, ...
            { "remove": { "id": value } }, ...
        ]
    },
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
    "expandPageSize": {"string": integer, ... },
    "attributes": [ string, ... ]
}
```

- `data` (optional)
  - contains the resource to be updated
  - specified attributes will **replace** the existing value
  - setting collection-based attributes this way may be an expensive operation; consider using "add" under such circumstances
  - attributes that are not specified will not be modified
- `update` (optional)
  - contains a list of (multiple) `add` and/or `remove` actions to be performed on the resources identified by the attribute
  - `add` (optional)
    - operates on collection-based attributes only
    - adds the resource with the specified ID to the collection
    - useful for linking Tags to Sections, for example
    - cannot be used for children of the resource being updated
  - `remove` (optional)
    - operates on collection-based attributes only
    - removed the resource with the specified ID from the collection
    - useful for unlinking Tags from Sections, for example
- `flags` (optional)
  - specifies one or more flags requesting additional information
  - `includeTotal`
    - retrieves the total amount of results
    - incurs a performance cost on the Documaster instance
    - defaults to false
- `expand` (optional)
  - specifies additional resources to be included in the response
- `expandPageSize` (optional)
  - specifies the page size for the additional resources in the `expand` parameter
- `attributes` (optional)
  - specifies the list of attributes to be returned for each resource in the response

At least one of `data` or `update` must be specified in an update request.

The same attribute cannot be updated via both `data` and `update`.

When both `data` and `update` have been specified, the attributes in the `data` element will be set first and the actions specified in `update` will be executed afterwards.

### Response

```
200 OK
Content-type: application/json

{
    "data": {
        attribute: value,
        ...,
        "resources": {
            "self": "{resource}/{id}",
            attribute: {
                "self": "{resource}/{id}/{related-resource}"
                "hasMore": boolean,
                "page": integer,
                "pageSize": integer
            },
            ...
        }
    },
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
    "expandPageSize": {"string": integer, ... },
    "attributes": [ string, ... ],   
}
```

- `data`
  - contains the updated resource
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `resources`
- `resources`
  - lists additional information per returned resource
  - `self`
    - contains a link to fetch the current resource
  - `attribute` (optional)
    - denotes a collection of related entities by their attribute
    - `self`
      - contains a link to fetch more of the related entities
    - `hasMore` (optional)
      - specifies whether more pages exist
      - only returned for related resources that represent a collection
    - `page` (optional)
      - the requested page
      - only returned for related resources that represent a collection
    - `pageSize` (optional)
      - the requested page size
      - only returned for related resources that represent a collection
- `flags`
  - the requested list of flags
- `expand` (optional)
  - the requested list of additional resources
- `expandPageSize` (optional)
  - the requested map of page sizes for the additional resources in the `expand` parameter
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `200 OK`
- `400 Bad request`
- `403 Unauthorized`

---

## Delete endpoint

Deletes the specified resource.

### Request

```
DELETE /{resource}/{id} HTTP/1.1
Authorization: Bearer {token}
```

### Response

```
204 No content
```

Response codes:
- `204 No content`
- `400 Bad request`
- `403 Unauthorized`

---

## Lookup endpoint

Looks up the Archive based on user-defined criteria using a [lookup query language](lookup-query-language.md).

### Request

```
POST /{resource}/lookup HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
Content-type: application/json

{
    "query": string,
    "parameters": { string: string, ... },
    "page": number,
    "pageSize" number,
    "sort": [ { "field": string, "order": "asc|desc"} , ... ],
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
    "expandPageSize": {"string": integer, ... },
    "attributes": [ string, ...]
}
```

- `query` (optional)
  - specifies a domain-specific query
- `parameters` (optional)
  - specifies a list of parameters and their values used in `query`
- `page` (optional)
  - specifies the page to fetch; 1-based
  - defaults to 1
- `pageSize` (optional)
  - specifies the page size
  - defaults to 10
- `sort` (optional)
  - specifies sort order to be applied on one or more attributes
  - an array of sort objects
- `flags` (optional)
  - specifies one or more flags requesting additional information
  - `includeTotal`
    - retrieves the total amount of results
    - incurs a performance cost on the Documsater instance
    - defaults to false
- `expand` (optional)
  - specifies additional resources to be included in the response
- `expandPageSize` (optional)
  - specifies the page size for the additional resources in the `expand` parameter
- `attributes` (optional)
  - specifies the list of attributes to be returned for each resource in the response

### Response

```
200 OK
Content-type: application/json

{
    "data": [ 
        {
            attribute: value,
            ...,
            "resources": {
                "self": "{resource}/{id}",
                attribute: {
                    "self": "{resource}/{id}/{related-resource}"
                    "hasMore": boolean,
                    "page": integer,
                    "pageSize": integer
                },
                ...
            }
        },
        ...
    ],
    "hasMore": boolean,
    "page": integer,
    "pageSize": integer,
    "sort": [ { "field": string, "order": "asc|desc"} , ... ],
    "flags": { "includeTotal": boolean },
    "expand": [ string, ... ],
    "expandPageSize": {"string": integer, ... },
    "attributes": [string, ...]
}
```

- `data`
  - contains a (potentially empty) list of the resources matching the specified lookup request
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `resources`
- `resources`
  - lists additional information per returned resource
  - `self`
    - contains a link to fetch the current resource
  - `attribute` (optional)
    - denotes a collection of related entities by their attribute
    - `self`
      - contains a link to fetch more of the related entities
    - `hasMore` (optional)
      - specifies whether more pages exist
      - only returned for related resources that represent a collection
    - `page` (optional)
      - the requested page
      - only returned for related resources that represent a collection
    - `pageSize` (optional)
      - the requested page size
      - only returned for related resources that represent a collection
- `hasMore`
  - specifies whether more pages exist
- `page`
  - the requested page
- `pageSize`
  - the requested page size
- `sort` (optional)
  - the specified sort order
  - an array of sort objects
- `flags`
  - the requested list of flags
- `expand` (optional)
  - the requested list of additional resources
- `expandPageSize` (optional)
  - the requested map of page sizes for the additional resources in the `expand` parameter
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `200 OK`
- `400 Bad request`
- `403 Unauthorized`

---

## Search endpoint

Performs search queries in the Documaster index based on a user-defined free-text query. Additional criteria can be specified using a [free-text search query language](free-text-search-query-language.md) to narrow down the results.

Note that the free-text search endpoints differs from the lookup endpoint in the following ways:

1. While the lookup endpoint will always operates on the **latest state** of the resources, the free-text search might not immediately reflect changes performed via the create, update, or delete endpoints. Ask us for more information on the delays when using the free-text search endpoint.

2. The free-text search endpoint allows you to perform non-exact search requests that will be sorted by relevance by default.

3. The full-text search is *likely* to perform better for heavily-filtered query requests.

4. The full-text search endpoint is supported only for the *document* and *entry* resources and document text queries are only supported for the *document* resource.

5. The internal index only contains information about the _last_ `document version` for every `document`. As a result, free-text searches can only be performed in the text of the latest document version, but not for earlier document versions.

### Request

```
POST /{resource}/search HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
Content-type: application/json
{
    "query": string,
    "queryAttributes": [ string, ...],
    "filterQuery": string,
    "parameters": { string: string, ... },
    "page": number,
    "pageSize" number,
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
    "expandPageSize": {"string": integer, ... },
    "attributes": [ string, ...]
}
```

- `query` (optional)
  - specifies a free-text query
  - any special characters would usually be escaped in the free-text query with the exception of quotes (") which allow you to perform "phrase searches"
- `queryAttributes` (optional)
  - specifies the attributes (own or related) on which to execute the free-text search query
  - allowed query attributes are text-based ones such as `title`, `description`, `text`, `fileName`, etc. but not date or numeric attributes such as `createdDate` or `versionNumber`, for example
- `filterQuery` (optional)
  - an additional (attribute-based) query to narrow down the results
  - uses the [free-text search query language](free-text-search-query-language.md)
- `parameters` (optional)
  - specifies a list of parameters and their values used in `filterQuery`
- `page` (optional)
  - specifies the page to fetch; 1-based
  - defaults to 1
- `pageSize` (optional)
  - specifies the page size
  - defaults to 10
- `flags` (optional)
  - specifies one or more flags requesting additional information
  - specifying the `enableAdvancedFTS` flag will enable advanced full-text search capabilities in the `query` field. See the `Advanced free-text-search` section below for more information
- `expand` (optional)
  - specifies additional resources to be included in the response
- `expandPageSize` (optional)
  - specifies the page size for the additional resources in the `expand` parameter
- `attributes` (optional)
  - specifies the list of attributes to be returned for each resource in the response

### Response

```
200 OK
Content-type: application/json

{
    "data": [ 
        {
            attribute: value,
            ...,
            "resources": {
                "self": "{resource}/{id}",
                attribute: {
                    "self": "{resource}/{id}/{related-resource}"
                    "hasMore": boolean,
                    "page": integer,
                    "pageSize": integer
                },
                ...
            }
        },
        ...
    ],
    "hasMore": boolean,
    "page": integer,
    "pageSize": integer,
    "total": integer,
    "flags": { },
    "expand": [ string, ... ],
    "expandPageSize": {"string": integer, ... },
    "attributes": [string, ...]
}
```

- `data`
  - contains a (potentially empty) list of the resources matching the specified free-text search request
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `resources`
- `resources`
  - lists additional information per returned resource
  - `self`
    - contains a link to fetch the current resource
  - `attribute` (optional)
    - denotes a collection of related entities by their attribute
    - `self`
      - contains a link to fetch more of the related entities
    - `hasMore` (optional)
      - specifies whether more pages exist
      - only returned for related resources that represent a collection
    - `page` (optional)
      - the requested page
      - only returned for related resources that represent a collection
    - `pageSize` (optional)
      - the requested page size
      - only returned for related resources that represent a collection
- `hasMore`
  - specifies whether more pages exist
- `page`
  - the requested page
- `pageSize`
  - the requested page size
- `total`
  - the total number of hits
- `flags`
  - the requested list of flags
- `expand` (optional)
  - the requested list of additional resources
- `expandPageSize` (optional)
  - the requested map of page sizes for the additional resources in the `expand` parameter
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `200 OK`
- `400 Bad request`
- `403 Unauthorized`

### Free-text search tips and tricks

We recommend taking the following precautions when performing free-text searches 

* Explicitly override the `documentVersions` `attribute`s to be returned in the response as the `text` attribute may unnecessarily increase the payload size significantly for documents that contain a lot of text.
* Consider making use of text highlight snippets (`highlights` returned for each resource) instead of retrieving the unabbreviated text attribute (`text`) that may be significantly larger.  
  See the model specification for more details about `highlights`.
* In order to receive any `highlights` for the attributes of a related resource, you must request that it is `expanded`. You would still get the primary results in the response even if you do not do this, but you will not receive the corresponding highlighted snippets.

Note that the API will always return the latest document version in document searches if you `expanded` the response with `documentVersions` in order to satisfy the most common use case for the search service - free-text searches in the document text.

### Advanced free-text search

By specifying the `enableAdvancedFTS` flag, certain characters will gain special meaning.

- Double quotes (") allow you to perform phrase searches for exact matches. For example: "This is my phrase search that requires an exact match"
- Asterisk (*) and question mark (?) allow you to perform wildcard searches.
  - Asterisks match zero or more consecutive characters at the beginning, middle, or end of a string. For example: \*master, Documas\*, Doc\*ter
  - Question marks match a single character at the beginning, middle, or end of a string. For example: ?ocumaster, Documaste?, Do?umaster
- Tilde (~) allows you to perform fuzzy search and proximity search.
  - When used with double-quotes ("phrase search"~3), it allows you to search for a phrase whose participating words are N words apart
  - When used with a single word (Kristian~), it allows you to search for a word with up to two edits to get a match. For example: Kristian~1 will also match Cristian, but Kristian~2 (which is equivalent to Kristian~) will also match Christian.
- Plus (+) allows you to request that a word or phrase appears in the resultset. For example: +Documaster or +"Documaster search"
- Minus (-) allows you to request that a word or phrase does not appear in the resultset. For example: -Documaster or -"Documaster search"
- Caret (^) allows you to boost a word or a phrase so that it gains a higher relevance score. For example: Documaster^1.3 or bug^0.3

---

## Upload endpoint

Uploads a single temporary document to Documaster as a stream.

### Request

```
POST /upload HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
Content-disposition: attachment; filename="ASCII_FILENAME"; filename*=utf-8''UTF8_FILENAME
Content-type: application/octet-stream

[Binary stream]
```

An uploaded temporary document will be removed from the server if it is not linked to a DocumentVersion resource within a specific amount of time.

The temporary document will not be available for download until it is linked to a DocumentVersion resource.

See [RFC 6266](http://tools.ietf.org/html/rfc6266) (description of the Content-Disposition header) and [RFC 5987](https://tools.ietf.org/html/rfc5987#section-3.2) (description of the encoding used in the filename* parameter).

The Content-Disposition header must specify at least one of the two parameters filename and filename*. When both are specified filename* takes precedence. The original charset of the filename* parameter value must be UTF-8, while the filename parameter value must be ASCII.

### Response

```
201 Created
Content-type: application/json

{
    "id": string,
    ...
}
```

Response codes:
- `201 Created`
- `400 Bad request`
- `403 Unauthorized`

---

## Download endpoint

Downloads a single document from Documaster as a stream.

### Request

```
GET /download/{id}
Authorization: Bearer {token}
```

### Response

```
200 OK
Content-disposition: attachment; filename="ASCII_FILENAME"; filename*=utf-8''UTF8_FILENAME
Content-type: */*

[Binary stream]
```

See [RFC 6266](http://tools.ietf.org/html/rfc6266) (description of the Content-Disposition header) and [RFC 5987](https://tools.ietf.org/html/rfc5987#section-3.2) (description of the encoding used in the filename* parameter).

---

# Access Group Endpoints

## Fetch all access groups endpoint

Retrieve the access groups, if any.

### Request

```
GET /access-group?page=1&pageSize=10 HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
```

- `page` (optional)
  - specifies the page to fetch; 1-based
  - defaults to 1
- `pageSize` (optional)
  - specifies the page size
  - defaults to 10
  - maximum allowed value is 1000

### Response

```
200 OK
Content-type: application/json

{
    "data": [
        {
            attribute: value,
            ...
        },
        ...
    ],
    "hasMore": boolean,
    "page": integer,
    "pageSize": integer
}
```

- `data`
  - contains a (potentially empty) list of access groups
- `hasMore`
  - specifies whether more pages exist
- `page`
  - the requested page
- `pageSize`
  - the requested page size

Response codes:
- `200 OK`
- `400 Bad request`

---

## Fetch my access groups endpoint

Retrieve the user access groups, if any.

### Request

```
GET /access-group/my?page=1&pageSize=10 HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
```

- `page` (optional)
  - specifies the page to fetch; 1-based
  - defaults to 1
- `pageSize` (optional)
  - specifies the page size
  - defaults to 10
  - maximum allowed value is 1000

### Response

```
200 OK
Content-type: application/json

{
    "data": [
        {
            attribute: value,
            ...
        },
        ...
    ],
    "hasMore": boolean,
    "page": integer,
    "pageSize": integer
}
```

- `data`
  - contains a (potentially empty) list of access groups
- `hasMore`
  - specifies whether more pages exist
- `page`
  - the requested page
- `pageSize`
  - the requested page size

Response codes:
- `200 OK`
- `400 Bad request`

---

## Fetch access group by ID endpoint

Retrieves the access group with the specified ID, if any.

### Request

```
GET /access-group/{id} HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
```

### Response

```
200 OK
Content-type: application/json

{
    "data": {
        attribute: value,
        ...
    }
}
```

- `data`
  - contains the (potentially empty) resource with the specified ID
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `resources`

Response codes:
- `200 OK`
- `400 Bad request`
- `403 Unauthorized`

---

## Create access group endpoint

Creates a new access group.

### Request

```
POST /access-group HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
Content-type: application/json

{
    "data": { attribute: value, ... }
}
```

- `data`
  - contains the access group to be created

### Response

```
201 Created
Content-type: application/json

{
    "data": {
        attribute: value,
        ...
    }
}
```

- `data`
  - contains the created resource

Response codes:
- `201 Created`
- `400 Bad request`
- `403 Unauthorized`

---

## Update access group endpoint

Updates an existing access group.

### Request

```
PUT /access-group/{id} HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
Content-type: application/json

{
    "data": { attribute: value, ... }
}
```

- `data` (optional)
  - contains the access group to be updated
  - specified attributes will **replace** the existing value

### Response

```
200 OK
Content-type: application/json

{
    "data": {
        attribute: value,
        ...
    }
}
```

- `data`
  - contains the updated resource

Response codes:
- `200 OK`
- `400 Bad request`
- `403 Unauthorized`

---

## Delete access group endpoint

Deletes the specified access group.

### Request

```
DELETE /access-group/{id} HTTP/1.1
Authorization: Bearer {token}
```

### Response

```
204 No content
```

Response codes:
- `204 No content`
- `400 Bad request`
- `403 Unauthorized`

---

# Error response format

The error response format will be returned anytime the server responds with a 4xx or 5xx status code.

Format:
```
{
    "errors": [
        {
            "errorId": string,
            "status": integer,
            "message": string,
            "path" string,
            "timestamp": timestamp
        },
        ...
    ]
}
```

- `errors`
  - contains a list of errors encountered by the server
  - `errorId`
    - A unique identifier of the error instance that could be reported to Documaster
  - `status`
    - the HTTP status code of the response
  - `message`
    - a human-readable error message
    - the message is not intended to be a unique identifier of the error that occurred
  - `path`
    - the resource path at which the error occurred
  - `timestamp`
    - the time at which the error occurred
