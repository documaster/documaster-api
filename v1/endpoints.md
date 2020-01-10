# Introduction

This document specifies information about the Documaster API endpoints:
- [Basic concepts](#basic-concepts)
- [Endpoints](#endpoints)
  - [Fetch endpoint](#fetch-endpoint) - GET /{resource}
  - [Fetch by ID endpoint](#fetch-by-id-endpoint) - GET /{resource}/{id}
  - [Create endpoint](#create-endpoint) - POST /{resource}
  - [Update endpoint](#update-endpoint) - PUT /{resource}
  - [Delete endpoint](#delete-endpoint) - DELETE /{resource}
  - [Lookup](#lookup-endpoint) - POST /{resource}/lookup
  - [Upload](#upload-endpoint) - POST upload-temp/
  - [Download](#download-endpoint) - GET download/{id}
- [Error response format](#error-response-format)

Refer to the [Model specification](model.md) for more information on the `{resource}` path parameter and the contents of `data`, `expand` and `attributes` used in this document.

Additionally, information about the [error response format](#error-response-format) can be found at the end of the document.

(!) Note that any users of the Documaster API must disregard the presense of unknown attributes, (nested) resources, or parameters in any of the endpoints to allow for backwards-compatible expansions of the API.

---

# Basic concepts

## Supported protocols

The web services support the following protocols:

- HTTPS

## Authentication and authorization

The web services require an access token (Bearer) to be specified in the HTTP headers when sending requests. A token can be obtained from the OpenID Connect Identity Provider used by the Documaster instance.

## HTTP status codes

The following table lists the HTTP status codes used by the web services.

| Status code                | Definition                                                                                               | Example                                                         |
|:---------------------------|:---------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------|
| 200 OK                     | The request was successful.                                                                              | HTTP/1.1 200 OK                                                 |
| 201 Created                | The request was fulfilled and resulted in a new resource being created.                                  | HTTP/1.1 201 Created                                            |
| 204 No Content             | The request was successful, but there is no content to send in the response payload body.                | HTTP/1.1 204 No Content                                         |
| 400 Bad Request            | The syntax of the request was invalid.                                                                   | HTTP/1.1 400 Bad Request                                        |
| 415 Unsupported Media Type | The payload format requested by the client is not supported by the server.                               | HTTP/1.1 415 Unsupported Media Type                             |
| 401 Unauthorized           | The request was denied due to an invalid or missing access token.                                        | <p>HTTP/1.1 401 Unauthorized</p><p>WWW-Authenticate: Bearer</p> |
| 403 Forbidden              | The request was denied due to the access token having insufficient privileges.                           | HTTP/1.1 403 Forbidden                                          |
| 404 Not Found              | The requested resource does not exist or is not accessible.                                              | HTTP/1.1 404 Not Found                                          |
| 409 Conflict               | Data was updated by another client after the last read by this client. The client may retry the request. | HTTP/1.1 409 Conflict                                           |
| 500 Internal Server Error  | An internal server error has occurred.                                                                   | HTTP/1.1 500 Internal Server Error                              |
| 503 Service Unavailable    | The service is temporarily unavailable.                                                                  | HTTP/1.1 503 Service Unavailable                                |
## Path

All endpoints appear under the following base path:

**{baseUrl}/api/v1/{resource}**

## Dates encoding

- Dates must be encoded as **ISO 8601** strings prior to sending them to the web services.
- Dates are returned by the web services as **ISO 8601** formatted strings.

---

# Endpoints

## Fetch endpoint

Retrieve the resources of the specified type, if any.

### Request

```
GET /{resource}?page=1&pageSize=10&flags=includeTotal&expand=resource1,resource2&attributes=field1,resource1.field1 HTTP/1.1
Accept: application/json
Authorization: Bearer ACCESS_TOKEN
```

- `page` (optional)
  - specifies the page to fetch; 1-based
  - defaults to 1
- `pageSize` (optional)
  - specifies the page size
  - defaults to 10
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

### Response

```
200 OK
Content-type: application/json

{
    "data": [
        {
            attribute: value,
            ...,
            "__resources": {
                "self": "resource/id",
                attribute: {
                    "self": "resource/id/attribute"
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
    "expand": [ string, ... ],
    "attributes": [string, ...],
}
```

- `data`
  - contains a (potentially empty) list of the resources matching the specified lookup request
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `__resources`
- `__resources`
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
GET /{resource}/{id}?expand=resource1,resource2&attributes=field1,resource1.field1 HTTP/1.1
Accept: application/json
Authorization: Bearer ACCESS_TOKEN
```

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
        "__resources": {
            "self": "resource/id",
            attribute: {
                "self": "resource/id/attribute"
                "hasMore": boolean,
                "page": integer,
                "pageSize": integer
            },
            ...
        }
    },
    "expand": [ string, ... ],
    "attributes": [string, ...],
}
```

- `data`
  - contains the (potentially empty) resource with the specified ID
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `__resources`
- `__resources`
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
- `expand` (optional)
  - the requested list of additional resources
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `200 OK`
- `400 Bad request`

---

## Create endpoint

Creates a new resource.

### Request

```
POST /{resource} HTTP/1.1
Accept: application/json
Authorization: Bearer ACCESS_TOKEN
Content-type: application/json

{
    "data": { attribute: value, ... },
    "expand": [ string, ... ],
    "attributes": [ string, ... ]
}
```

- `data`
  - contains the resource to be created
  - at most one thousand (1000) nested resources in total can be created as part of the same request
- `expand` (optional)
  - specifies additional resources to be included in the response
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
        "__resources": {
            "self": "resource/id",
            attribute: {
                "self": "resource/id/attribute"
                "hasMore": boolean,
                "page": integer,
                "pageSize": integer
            },
            ...
        }
    },
    "expand": [ string, ... ],
    "attributes": [ string, ... ]
}
```

- `data`
  - contains the created resource
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `__resources`
- `__resources`
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
- `expand` (optional)
  - the requested list of additional resources
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `201 Created`
- `400 Bad request` 

---

## Update endpoint

Updates an existing resource.

### Request

```
PUT /{resource}/{id} HTTP/1.1
Accept: application/json
Authorization: Bearer ACCESS_TOKEN
Content-type: application/json

{
    "data": { attribute: value, ... },
    "update": {
        attribute: [
            { "add": { attribute: value } }, ...
            { "remove": { attribute: value } }, ...
            { "set": { attribute: value } }, ...
        ]
    },
    "expand": [ string, ... ],
    "attributes": [ string, ... ]
}
```

- `data` (optional)
  - contains the resource to be updated
  - specified attributes will **replace** the existing value
  - attributes that are not specified will not be modified
- `update` (optional)
  - contains a list of (multiple) `add`, `set`, or `remove` actions to be performed on the objects behind an attribute
  - `add` (optional)
    - operates on collection-based attributes only
    - adds the specified resource to the collection
    - useful for linking Tags to Sections, for example
  - `remove` (optional)
    - operates on collection-based attributes only
    - removed the specified resource from the collection
    - useful for unlinking Tags fron Sections, for example
  - `set` (optional)
    - operates both on collection-based attributes and on single-value attributes
    - replaces (sets) the attribute value with the specified one
- `expand` (optional)
  - specifies additional resources to be included in the response
- `attributes` (optional)
  - specifies the list of attributes to be returned for each resource in the response

At least one of `data` or `update` must be specified in an update request.

### Response

```
200 OK
Content-type: application/json

{
    "data": {
        attribute: value,
        ...,
        "__resources": {
            "self": "resource/id",
            attribute: {
                "self": "resource/id/attribute"
                "hasMore": boolean,
                "page": integer,
                "pageSize": integer
            },
            ...
        }
    },
    "expand": [ string, ... ],
    "attributes": [ string, ... ],   
}
```

- `data`
  - contains the updated resource
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `__resources`
- `__resources`
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
- `expand` (optional)
  - the requested list of additional resources
- `attributes` (optional)
  - the requested list of attributes

Response codes:
- `200 OK`
- `400 Bad request` 
- `404 Not found`

---

## Delete endpoint

Deletes the specified resource.

### Request

```
DELETE {resource}/{id} HTTP/1.1
Authorization: Bearer ACCESS_TOKEN
```

### Response

```
204 No content
```

Response codes:
- `204 No content`
- `400 Bad request`
- `404 Not found`

---

## Lookup endpoint

Looks up the Archive based on user-defined criteria using a [query language](query-language.md).

### Request

```
POST /{resource}/lookup HTTP/1.1
Accept: application/json
Authorization: Bearer ACCESS_TOKEN
Content-type: application/json

{
    "query": string,
    "parameters": { string: string, ... },
    "page": number,
    "pageSize" number,
    "sort": { string: asc|desc, ... },
    "flags": { "includeTotal": boolean }
    "expand": [ string, ... ],
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
- `flags` (optional)
  - specifies one or more flags requesting additional information
  - `includeTotal`
    - retrieves the total amount of results
    - incurs a performance cost on the Documsater instance
    - defaults to false
- `expand` (optional)
  - specifies additional resources to be included in the response
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
            "__resources": {
                "self": "resource/id",
                attribute: {
                    "self": "resource/id/attribute"
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
    "sort": { string: asc|desc, ... },
    "flags": { "includeTotal": boolean },
    "expand": [ string, ... ],
    "attributes": [string, ...]
}
```

- `data`
  - contains a (potentially empty) list of the resources matching the specified lookup request
  - at most 10 nested resources per type will be returned; more information on retrieving additional resources can be found in `__resources`
- `__resources`
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

## Upload endpoint

Uploads a single temporary document to Documaster as a stream.

### Request

```
POST /upload HTTP/1.1
Accept: application/json
Authorization: Bearer ACCESS_TOKEN
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
200 OK
Content-type: application/json

{
    "data": {
        "id": string,
    }
}
```

---

## Download endpoint

Downloads a single document from Documaster as a stream.

### Request

```
GET download/{id}
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

# Error response format

The error response format will be returned anytime the server responds with a 4xx or 5xx status code.

Format:
```
{
    "errors": [
        {
            "status": integer,
            "message": string
        },
        ...
    ]
}
```

- `errors`
  - contains a list of errors encountered by the server
  - `status`
    - the HTTP status code of the response
  - `message`
    - an error message 