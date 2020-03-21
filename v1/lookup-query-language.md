Lookup Query language
-------

## Overview

The web services support a simple but powerful lookup query language that allows one to query for resources of a particular type, specifying **criteria** on the **resource attributes** as well as **criteria on the attributes of linked resources**. Note that a *linked resource* may be linked directly or indirectly (via another resource).

## Attributes

The metadata model exposed by the web services defines which attributes can be queried. The following table shows examples of how you can use attributes when querying for objects:

| Query for                                               | What attribute to use | Comment                                                                                                                   |
|---------------------------------------------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------|
| **Entry** with a specific **id**                        | id                    |                                                                                                                           |
| **Entry** with a specific **title**                     | title                 | Note that depending on the type of the attribute, one could use more advanced tools such as "contains" or range criteria. |
| **Entry** in a specific **Section**                     | section.id            | The **id** attribute of **Section** can be used.                                                                          |
| **Entry** in a **Section** with a specific **title**    | section.title         | Chaining via attributes allows us to specify criteria on attributes in linked resources.                                  |
| **Document** in a **Section** with a specific **title** | entry.section.title   | More that one level of chaining is allowed.                                                                               |

## Expressions

An expression consists of an *attribute*, an *operator*, and one or two *parameters*.

Parameter values are not a part of the query and must match the data type of the attribute for which a criterion is specified.

### Simple expressions

The following table shows the supported simple expressions:

| Expression           | Supported attribute data types | Meaning                                                            | Comment                                                     |
|----------------------|--------------------------------|--------------------------------------------------------------------|-------------------------------------------------------------|
| attribute = @param1  | string, integer                | An attribute is equal to the value of the specified parameter.     |                                                             |
| attribute != @param2 | string, integer                | An attribute is not equal to the value of the specified parameter. |                                                             |
| attribute %= @param3 | string                         | An attribute is like the value of the specified parameter.         | A % sign in the parameter value denotes *any character(s)*. |

### Range expressions

The following table shows the supported range expressions:

| Expression                    | Supported attribute data types | Meaning                                                                                  | Comment                                                                  |
|-------------------------------|--------------------------------|------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| attribute = [@param1:@param2] | date, number                   | The value of an attribute is within the (inclusive) range defined by the two parameters. | A left/right square bracket defines the start/end of an inclusive range. |
| attribute = {@param1:@param2} | date, number                   | The value of an attribute is within the (exclusive) range defined by the two parameters. | A left/right curly bracket defines the start/end of an exclusive range.  |
| attribute = [@param1:@param2} | date, number                   | The value of an attribute is within the range defined by the two parameters.             | Start is inclusive, end is exclusive.                                    |
| attribute = {@param1:@param2] | date, number                   | The value of an attribute is within the range defined by the two parameters.             | Start is exclusive, end is inclusive.                                    |

Note that a ***** character in the parameter value denotes *any value*.


## Grouping expressions

Expressions can be grouped using boolean operators, which are listed in the following table:

| Operator     | Meaning     | Precedence | Example                              |
|--------------|-------------|------------|--------------------------------------|
| &amp;&amp;   | Logical AND | 1st        | expression1 &amp;&amp; expression2   |
| &vert;&vert; | Logical OR  | 2nd        | expression1 &vert;&vert; expression2 |

The order of precedence can be changed using parentheses:

| Operator | Meaning                                             | Example                                               |
|----------|-----------------------------------------------------|-------------------------------------------------------|
| ( )      | Change the order of precedence of boolean operators | (expression1 &vert;&vert; expression2) && expression3 |

## Expressions referencing related resources

As already exhibited in some of the examples above you can reference attributes of related resources in your queries by chaining references. For example, when looking up a document you can write the following query:

```
entry.section.title = @sectionTitle && entry.section.classification.id = @classificationId
```

There are a total of three ways that you can write similar queries:

### 1. Plain reference chaining

With plain reference chaining you simply chain the reference attributes until you reach the resource attribute you want to query on:
```
entry.section.title = @sectionTitle && entry.section.classification.id = @classificationId
```

This is the simplest and most intuitive approach and the one that you are likely to use in the majority of your queries.

### 2. Aliased reference chaining

With aliased reference chaining you gain more control over queries that contain expressions on multiple attributes of the same related resource at the cost of introducing some additional complexity. Let us explaing this with an example.

Suppose you want to find an Entry that belongs to a Section and is linked to two Tags with different IDs. You would be inclined to write the following query __which would not produce any results__:

```
section.id = @sectionId && tags.id = @tagId1 && tags.id = @tagId2
```

As already suggested, the query will always return 0 results. The reason why this happens is that with plain reference chaining each reference is always considered to pertain to a single related entity. As such, the `tags` above would be mapped to a single related tag by the query parser. As you can probably already guess, a Tag has only one unqiue identifier (id) and the two statements in the example above (`tags.id = @tagId1` and `tags.id = @tagId2`) essentially cancel each other out.

To signal to the query parser that you want these references to be treated as references to two __distinct__ related Tags, you can _alias_ (and re-use) your references. With aliased reference chaining, the query will be re-written to:

```
section.id = @sectionId && tags#myTag1.id = @tagId1 && tags#myTag2.id = @tagId2
```

Pay special attention ot the addition of the `#myTag1` and `#myTag2` aliases assigned to each of the `tags` references. This will signal to the query parser that the two references refer to two __distinct__ related tags and your query will now have the desired behavior.

### 3. Mixed reference chaining

Your third and last option is to use a mix of _Plain reference chaining_ and _Aliased reference chaining_ as has already been hinted above:

```
section.id = @sectionId && tags#myTag1.id = @tagId1 && tags#myTag2.id = @tagId2
```

Users are allowed to use both the default (and simpler) plain reference chaining (`section`) and the more complex (and more powerful) mixed reference chaininig (`tags#myTag1`) in the same query.

## Examples

| Search for                                                             | Query                                | Parameter value                                        | Comment                                                    |  |
|------------------------------------------------------------------------|--------------------------------------|--------------------------------------------------------|------------------------------------------------------------|--|
| **Entry** with a given **id**                                          | id = @id                             | 100                                                    | 100 is the **id** of the **Entry**                         |  |
| **Entry** with an exact **title**                                      | title = @title                       | "Invoice"                                              |                                                            |  |
| **Entry** with a **title** not equal to                                | title != @title                      | "Invoice"                                              |                                                            |  |
| **Entry** with a **title** that starts with                            | title %= @title                      | "Inv%"                                                 |                                                            |  |
| **Entry** with a **title** that ends with                              | title %= @title                      | "%voice"                                               |                                                            |  |
| **Entry** with a **title** that contains a string                      | title %= @title                      | "%nvo%"                                                |                                                            |  |
| **Entry** with a **createdDate** within a particular range (inclusive) | createdDate=[@start:@end]            | <p>2019-01-01T00:00:00Z</p><p>2019-01-01T23:59:59Z</p> | **Entries** created on 01/01/2019                          |  |
| **Entry** with a **createdDate** within a particular range (exclusive) | createdDate={@start:@end}            | <p>2019-01-01T00:00:00Z</p><p>2019-01-02T00:00:00Z</p> | **Entries** created on 01/01/2019                          |  |
| **Entry** in a specific **Section**                                    | section.id = @sectionId              | 10                                                     | 10 is the **id** of the **Section**                        |  |
| **Entry** with a specific **ExternalId**                               | externalIds.externalId = @externalId | e01d37bb-a2dc-4516-a5df-eb502fdd3f3510                 | e01d37bb-a2dc-4516-a5df-eb502fdd3f35 is the **externalId** |  |
| **Entry** linked to two distinct **Tags**                              | tags#1.id = @id1 && tags#2.id = @id2 | <p>1234</p><p>5678</p>                                 |                                                            |  |
| **Document** in a specific **Section**                                 | entry.section.id = @sectionId        | 5                                                      | 5 is the **id** of the **Section**                         |  |
| **Document** in a **Section** with a specific **title**                | entry.section.title = @title         | "Employees"                                            | &nbsp;                                                     |  |
