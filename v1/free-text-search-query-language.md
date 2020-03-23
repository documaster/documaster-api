Free-text search query language
-------

## Overview

The web services support a simple but powerful free-text search language that allows one to search both in the metadata and content of documents uploaded to Documaster. Additional **criteria** can be specified on the **resource attributes** or on the **attributes of linked resources** to narrow down the results. Note that a *linked resource* may be linked directly or indirectly (via another resource).

## Attributes

The metadata model exposed by the web services defines which attributes can be searched for and used for filtering. The following table shows examples of how you can use attributes when performing free-text search for objects:

| Query for                                               | What attribute to use | Comment                                                                                                     |
|---------------------------------------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------|
| **Entry** with a specific **id**                        | id                    |                                                                                                             |
| **Entry** with a specific **title**                     | title                 | Note that depending on the type of the attribute, one could use more advanced tools such as range criteria. |
| **Entry** in a specific **Section**                     | section.id            | The **id** attribute of **Section** can be used.                                                            |
| **Entry** in a **Section** with a specific **title**    | section.title         | Chaining via attributes allows us to specify criteria on attributes in linked resources.                    |
| **Document** in a **Section** with a specific **title** | entry.section.title   | More that one level of chaining is allowed.                                                                 |

## Expressions

An expression consists of an *attribute*, an *operator*, and one or two *parameters*.

Parameter values are not a part of the query and must match the data type of the attribute for which a criterion is specified.

### Simple expressions

The following table shows the supported simple expressions:

| Expression           | Supported attribute data types | Meaning                                                            | Comment                                                     |
|----------------------|--------------------------------|--------------------------------------------------------------------|-------------------------------------------------------------|
| attribute = @param1  | string, integer                | An attribute is equal to the value of the specified parameter.     |                                                             |
| attribute != @param2 | string, integer                | An attribute is not equal to the value of the specified parameter. |                                                             |

### Range expressions

The following table shows the supported range expressions:

| Expression                    | Supported attribute data types | Meaning                                                                                  | Comment                                                                  |
|-------------------------------|--------------------------------|------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| attribute = [@param1:@param2] | date, number                   | The value of an attribute is within the (inclusive) range defined by the two parameters. | A left/right square bracket defines the start/end of an inclusive range. |
| attribute = {@param1:@param2} | date, number                   | The value of an attribute is within the (exclusive) range defined by the two parameters. | A left/right curly bracket defines the start/end of an exclusive range.  |
| attribute = [@param1:@param2} | date, number                   | The value of an attribute is within the range defined by the two parameters.             | Start is inclusive, end is exclusive.                                    |
| attribute = {@param1:@param2] | date, number                   | The value of an attribute is within the range defined by the two parameters.             | Start is exclusive, end is inclusive.                                    |

Note that a __*__ character in the parameter value denotes *any value*.

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

## Examples

| Search for                                                             | Query                                | Parameter value                                        | Comment                                                    |  |
|------------------------------------------------------------------------|--------------------------------------|--------------------------------------------------------|------------------------------------------------------------|--|
| **Entry** with a given **id**                                          | id = @id                             | 100                                                    | 100 is the **id** of the **Entry**                         |  |
| **Entry** with an exact **title**                                      | title = @title                       | "Invoice"                                              |                                                            |  |
| **Entry** with a **title** not equal to                                | title != @title                      | "Invoice"                                              |                                                            |  |
| **Entry** with a **createdDate** within a particular range (inclusive) | createdDate=[@start:@end]            | <p>2019-01-01T00:00:00Z</p><p>2019-01-01T23:59:59Z</p> | **Entries** created on 01/01/2019                          |  |
| **Entry** with a **createdDate** within a particular range (exclusive) | createdDate={@start:@end}            | <p>2019-01-01T00:00:00Z</p><p>2019-01-02T00:00:00Z</p> | **Entries** created on 01/01/2019                          |  |
| **Entry** in a specific **Section**                                    | section.id = @sectionId              | 10                                                     | 10 is the **id** of the **Section**                        |  |
| **Entry** with a specific **ExternalId**                               | externalIds.externalId = @externalId | e01d37bb-a2dc-4516-a5df-eb502fdd3f3510                 | e01d37bb-a2dc-4516-a5df-eb502fdd3f35 is the **externalId** |  |
| **Document** in a specific **Section**                                 | entry.section.id = @sectionId        | 5                                                      | 5 is the **id** of the **Section**                         |  |
| **Document** in a **Section** with a specific **title**                | entry.section.title = @title         | "Employees"                                            | &nbsp;                                                     |  |
