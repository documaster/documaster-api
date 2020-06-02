# Introduction

This document lists common requests and responses using the Documaster API.

The examples are based on the (more formal) [Model specification](model.md) and [Endpoints specification](endpoints.md).

You can jump right to the section you are interested in:
- [Upload examples](#upload)
- [Download examples](#download)
- [Classification examples](#classification)
- [Tag examples](#tag)
- [Section examples](#section)
- [Entry examples](#entry)
- [Search examples](#search)

---

# Examples

## Upload

Uploads a temporary document to Documaster.

Request:
```
POST /upload-temp HTTP/1.1
Accept: application/json
Authorization: Bearer {token}
Content-disposition: attachment; filename="filename.txt"; filename*=utf-8''filename.txt
Content-type: application/octet-stream

[Binary stream]
```

Response:
```
200 OK
Content-type: application/json

{
    "data": {
        "id": "db7c7593-32d3-4298-bbf7-681237bd58ef",
    }
}
```

## Download

Downloads a document from Documaster.

Request:
```
GET /download/db7c7593-32d3-4298-bbf7-681237bd58ef
Authorization: Bearer {token}
```

Response:
```
200 OK
Content-disposition: attachment; filename="filename.txt"; filename*=utf-8''filename.txt
Content-type: */*

[Binary stream]
```

## Classification

### Create

Create a Classification with title "Employees".

Request:
```
POST /classification HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "data": {
        "title": "Employees"
    }
}
```

Response:
```
201 Created

{
    "data": {
        "id": "58dc1089-de65-4b98-a1f6-2f162c21dd05",
        "revision": 1,
        "title": "Employees",
        "resources": {
            "self": "classification/58dc1089-de65-4b98-a1f6-2f162c21dd05"
        }
    }
}
```

### Create with tags

Create a Classification with title "Employees" and two Tags.

Request:
```
POST /classification HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "data": {
        "title": "Employees",
        "tags": [
            {"title": "John Doe"},
            {"title": "Jane Doe"}
        ]
    },
    "expand": ["tags"]
}
```

Response:
```
201 Created
Content-type: application/json

{
    "data": {
        "id": "58dc1089-de65-4b98-a1f6-2f162c21dd05",
        "revision": 1,
        "title": "Employees",
        "tags": {
            {
                "id": "f140f916-33d8-41e4-abfa-a65347df7930",
                "revision": 1,
                "title": "John Doe",
                "classification": {"id": "58dc1089-de65-4b98-a1f6-2f162c21dd05"}
            },
            {
                "id": "40569fbd-e4a3-49e4-97ce-fc6e3bb9dadf",
                "revision": 1,
                "title": "Jane Doe",
                "classification": {"id": "58dc1089-de65-4b98-a1f6-2f162c21dd05"}
            }
        },
        "resources": {
            "self": "classification/58dc1089-de65-4b98-a1f6-2f162c21dd05",
            "tags": {
                "self": "classification/58dc1089-de65-4b98-a1f6-2f162c21dd05/tags"
                "hasMore": false,
                "page": 0,
                "pageSize": 10
            }
        }
    },
    "expand": ["tags"]
}
```

---

## Tag

### Create

Create a tag with title "John Doe" and link it to a Classification.

Request:
```
POST /tag HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "data": {
        "title": "John Doe",
        "classification": {"id": "58dc1089-de65-4b98-a1f6-2f162c21dd05"}
    }
}
```

Response:
```
201 Created
Content-type: application/json

{
    "data": {
        "id": "",
        "revision": 1,
        "title": "John Doe",
        "classification": {"id": "58dc1089-de65-4b98-a1f6-2f162c21dd05"},
        "resources": {
            "self": "classification/58dc1089-de65-4b98-a1f6-2f162c21dd05/tag/645fe6ed-2c50-4fc2-ac37-8320f3b5eff0",
            "classification": {
                "self": "classification/58dc1089-de65-4b98-a1f6-2f162c21dd05"
            }
        }
    }
}
```

### Delete

Delete a tag.

Request:
```
DELETE /tag/{id}
Authorization: Bearer {token}
Content-type: application/json
```

Response:
```
204 No content
```

### Look up tags by title

Look up Tags with specific titles.

Request:
```
POST /tag/lookup HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "query": "title=@title1 || title=@title2",
    "parameters": {"@title1": "John Doe", "@title2": "Jane Doe"},
    "page": 1,
    "pageSize": 3,
    "sort": {"field": "title", "order": "asc"}
}
```

Response:
```
200 OK

{
    "data": [
        {
            "id": "bfe49142-895e-4db9-9255-d83d615848a9",
            "revision": 1,
            "title": "John Doe",
            "classification": {"id": "0e06cfd4-722b-4c14-aab3-1753e5df5761"},
            "resources": {
                "self": "tag/bfe49142-895e-4db9-9255-d83d615848a9",
                "classification": {
                    "self": "classification/0e06cfd4-722b-4c14-aab3-1753e5df5761",
                }
            }
        },
        {
            "id": "3ac6c707-d3d8-47f0-b3c5-53832dd4a83f",
            "revision": 1,
            "title": "Jane Doe",
            "classification": {"id": "0e06cfd4-722b-4c14-aab3-1753e5df5761"},
            "resources": {
                "self": "tag/3ac6c707-d3d8-47f0-b3c5-53832dd4a83f",
                "classification": {
                    "self": "classification/0e06cfd4-722b-4c14-aab3-1753e5df5761",
                }
            }
        }
    ],
    "hasMore": false,
    "page": 1,
    "pageSize": 3,
    "sort": {"field": "title", "order": "asc"},
}
```

---

## Section

### Create

Create a section and link it to two Classifications.

Request:
```
POST /section HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "data": {
        "title": "HR",
        "classifications": [
            {"id": "58dc1089-de65-4b98-a1f6-2f162c21dd05"},
            {"id": "1f858f70-aaee-49bb-a869-4716f91c41dc"}
        ]
    }
}
```

Response:
```
201 Created
Content-type: application/json

{
    "data": {
        "id": "75c4c084-b790-480b-bc00-0aa4698f90ac",
        "revision": 1,
        "title": "HR",
        "resources": {
            "self": "section/75c4c084-b790-480b-bc00-0aa4698f90ac"
        }
    }
}
```

---

## Entry

### Create

Create an entry with title "Invoice #18726182553" and tag it with two Tags.

Request:
```
POST /entry HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "data": {
        "title": "Invoice #18726182553",
        "section": {"id": "bf3d420f-797a-4629-9274-3ad0ba80ec9d"},
        "tags": [
            {"id": "f140f916-33d8-41e4-abfa-a65347df7930"},
            {"id": "40569fbd-e4a3-49e4-97ce-fc6e3bb9dadf"}
        ]
    },
    "expand": ["tags"]
}
```

Response:
```
201 Created
Content-type: application/json

{
    "data": {
        "id": "55cba765-ea62-48ca-9c81-9ca77e2275a1",
        "revision": 1,
        "title": "Invoice #18726182553", 
        "section": {"id": "bf3d420f-797a-4629-9274-3ad0ba80ec9d"},
        "tags": [
            {
                "id": "136aa987-68c6-4b99-a976-84a4076f825f",
                "revision": 3,
                "title": "Supplier invoice",
                "classification": {"id": "1fe22567-10b5-4bfd-be57-7f6cca83a927"}
            },
            {
                "id": "c35d11b1-2463-4614-9168-7e9778faf4e1",
                "revision": 2,
                "title": "Documaster",
                "classification": {"id": "d3a676f0-ea36-47b6-909b-361dbaade968"}
            }
        ],
        "resources": {
            "self": "entry/55cba765-ea62-48ca-9c81-9ca77e2275a1",
            "section": {
                "self": "section/bf3d420f-797a-4629-9274-3ad0ba80ec9d",
            },
            "tags": {
                "self": "entry/55cba765-ea62-48ca-9c81-9ca77e2275a1/tag",
                "hasMore": false,
                "page": 1,
                "pageSize": 2,
            }
        }
    }
}
```

### Create (advanced)

Create an Entry and:
- tag it with two Tags;
- label it with an External ID;
- create two Documents labeling them with External IDs;
- create two Parties.

Request:
```
POST /entry HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "data": {
        "title": "Invoice #18726182553",
        "section": {"id": "bf3d420f-797a-4629-9274-3ad0ba80ec9d"},
        "tags": [
            {"id": "f140f916-33d8-41e4-abfa-a65347df7930"},
            {"id": "40569fbd-e4a3-49e4-97ce-fc6e3bb9dadf"}
        ],
        "documents": [
            {
                "title": "Invoice #18726182553", 
                "documentVersions": {
                    "electronicDocument": {"id": "db7c7593-32d3-4298-bbf7-681237bd58ef"},
                    "externalIds": [{"externalId": "{CRM Invoice Identifier}"}]
                }
            },
            {
                "title": "Receipt", 
                "documentVersions": {
                    "electronicDocument": {"id": "feb812d9-f689-494f-960e-2f604c3dc2e8"},
                    "externalIds": [{"externalId": "CRM-system-id-872563"}]
                }
            }
        ],
        "parties": [
            {"name": "John Doe", "address": "Oslo"},
            {"name": "Jane Doe", "address": "Trondheim"}
        ],
        "externalIds": [
            {"externalId": "CRM-system-id-872564"}
        ]
    },
    "expand": ["parties"],
    "attributes": ["parties.id"]
}
```

Response:
```
201 Created
Content-type: application/json

{
    "data": {
        "id": "55cba765-ea62-48ca-9c81-9ca77e2275a1",
        "revision": 1,
        "title": "Invoice #18726182553", 
        "section": {"id": "bf3d420f-797a-4629-9274-3ad0ba80ec9d"},
        "parties": [
            {"id": "cf3995b7-9614-4cf4-b264-887febf92f84"},
            {"id": "870769f1-9524-47a7-bb2e-afdfab167a0c"}
        ],
        "resources": {
            "self": "entry/55cba765-ea62-48ca-9c81-9ca77e2275a1",
            "section": {
                "self": "section/bf3d420f-797a-4629-9274-3ad0ba80ec9d",
            },
            "parties": {
                "self": "entry/55cba765-ea62-48ca-9c81-9ca77e2275a1/party",
                "hasMore": false,
                "page": 1,
                "pageSize": 2,
            }
        }
    },
    "expand": ["parties"],
    "attributes": ["parties.id"]
}
```

### Update

Update the title of an Entry and move it to a different Section without changing anything else.

Request:
```
PUT /entry HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "data": {
        "id": "55cba765-ea62-48ca-9c81-9ca77e2275a1",
        "revision": 1,
        "title": "Changed invoice number",
        "section": {"id": "c2024523-3b13-4ccf-bcdf-e0746d1ed477"}
    }
}
```

Response:
```
200 OK
Content-type: application/json

{
    "data": {
        "id": "55cba765-ea62-48ca-9c81-9ca77e2275a1",
        "revision": 2,
        "title": "Changed invoice number", 
        "section": {"id": "c2024523-3b13-4ccf-bcdf-e0746d1ed477"},
        "resources": {
            "self": "entry/55cba765-ea62-48ca-9c81-9ca77e2275a1",
            "section": {
                "self": "section/bf3d420f-797a-4629-9274-3ad0ba80ec9d",
            }
        }
    }
}
```

### Update (advanced)

Update an Entry by:
- tagging it with one more Tag;
- untagging it from a Tag;
- moving it to a different Section;
- updating its title;
- replacing all External IDs;
- deleting a Document.

All other fields will be left intact meaning:
- previously linked Tags will remain the same;
- previously created Parties will remain the same;
- unmodified attributes will remain the same.

Request:
```
PUT /entry/{id} HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "data": {
        "id": "55cba765-ea62-48ca-9c81-9ca77e2275a1",
        "revision": 1,
        "title": "Changed invoice number",
        "section": {"id": "c2024523-3b13-4ccf-bcdf-e0746d1ed477"}
    },
    "update": {
        "tags": [
            {"add": {"id": "f140f916-33d8-41e4-abfa-a65347df7930"}},
            {"remove": {"id": "52fc6339-80ec-456c-b8dc-05e0f569d539"}}
        ],
        "documents": [
            {"remove": {"id": "ad5524ba-87b6-4a80-949f-15a7176a5dbd"}}
        ],
        "externalIds": [
            {"set": [{"externalId": "New external identifier"}]}
        ]
    }
}
```

Response:
```
200 OK

{
    "data": [
        {
            "id": "55cba765-ea62-48ca-9c81-9ca77e2275a1",
            "revision": 2,
            "title": "Changed invoice number", 
            "description": "Invoice description",
            "section": {"id": "c2024523-3b13-4ccf-bcdf-e0746d1ed477"},
            "resources": {
                "self": "entry/55cba765-ea62-48ca-9c81-9ca77e2275a1",
                "section": {
                    "self": "section/c2024523-3b13-4ccf-bcdf-e0746d1ed477",
                }
            }
        }
    ]
}
```

### Delete

Delete the entry with the specified ID.

Request:
```
DELETE /entry/55cba765-ea62-48ca-9c81-9ca77e2275a1 HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json
```

Response:
```
204 No content
```

### Look up by External ID

Look up an Entry with a specific External ID.

Request:
```
POST /entry/lookup HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "query": "externalIds.externalId=@externalId",
    "parameters": {"@externalId": "CRM-system-862162"},
    "page": 1,
    "pageSize": 10,
    "sort": {"field": "title", "order": "asc"},
    "flags": {"includeTotal": true},
    "expand": ["tags", "tags.classification"],
    "attributes: ["tags.title", "tags.classification.title"],
}
```

Response:
```
200 OK

{
    "data": [
        {
            "id": "55cba765-ea62-48ca-9c81-9ca77e2275a1",
            "revision": 1,
            "title": "Invoice #18726182553",
            "section": {"id": "c2024523-3b13-4ccf-bcdf-e0746d1ed477"},
            "tags": [
                {"title": "John Doe", "classification": {"title": "Employees"}},
                {"title": "Jane Doe", "classification": {"title": "Employees"}}
            ],
            "resources": {
                "self": "entry/55cba765-ea62-48ca-9c81-9ca77e2275a1",
                "section": {
                    "self": "section/c2024523-3b13-4ccf-bcdf-e0746d1ed477",
                },
                "tags": {
                    "self": "entry/55cba765-ea62-48ca-9c81-9ca77e2275a1/tag",
                    "hasMore": false,
                    "page": 1,
                    "pageSize": 2
                }
            }
        }
    ],
    "hasMore": false,
    "total": 1,
    "page": 1,
    "pageSize": 10,
    "sort": {"field": "title", "order": "asc"},
    "flags": {"includeTotal": true},
    "expand": ["tags", "tags.classification"],
    "attributes: ["tags.title", "tags.classification.title"],
}
```

---

## Search

### Free-text document search

Search for documents that contain user-specified free text and sort them by relevance.

The query is further narrowed down to documents annotated with specific Tags and belonging to a specific Section.

Request:
```
POST /document/search HTTP/1.1
Authorization: Bearer {token}
Content-type: application/json

{
    "query": "Termination date",
    "queryAttributes": ["documentVersions.text"],
    "filterQuery": "entry.section.title=@sectionTitle && (entry.tags.title=@tagTitle1 || entry.tags.title=@tagTitle2)",
    "parameters": {"@sectionTitle": "HR", "@tagTitle1": "Contracts", "@tagTitle2": "John Doe"},
    "page": 1,
    "pageSize": 10,
    "expand": ["documentVersions", "entry"],
    "attributes": ["title", "documentVersions.text", "entry.title"]
}
```

Response:
```
200 OK

{
    "data": [
        {
            "id": "bfe49142-895e-4db9-9255-d83d615848a9",
            "title": "John Doe Employment Contract",
            "entry": {
                "id": "0e06cfd4-722b-4c14-aab3-1753e5df5761",
                "title": "John Doe Employment Contract"
            },
            "documentVersions": [
                {
                    "id": "ae07df14-731b-12ab-ca13-2c54e5df5762",
                    "text": "[shortened for brevity] Date of termination: 14/02/2001 [shortened for brevity]",
                    "highlights": {"text": ["[shortened for brevity] ... |=hlstart=|Date|=hlstop=| of |=hlstart=|termination|=hlstop=| ... [shortened for brevity]"]}
                }
            ],
            "resources": {
                "self": "document/bfe49142-895e-4db9-9255-d83d615848a9",
                "entry": {
                    "self": "entry/0e06cfd4-722b-4c14-aab3-1753e5df5761",
                },
                "documentVersions": {
                    "self": "document/bfe49142-895e-4db9-9255-d83d615848a9/document-versions",
                    "hasMore": false,
                    "page": 1,
                    "pageSize": 2
                }
            }
        }
    ],
    "hasMore": false,
    "total": 1,
    "page": 1,
    "pageSize": 10,
    "expand": ["documentVersions", "entry"],
    "attributes": ["title", "documentVersions.text", "entry.title"]
}
```