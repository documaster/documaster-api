# Table of contents

- [Examples](examples.md)
- [Data model](model.md)
- [API endpoints](endpoints.md)
- [Access control](access-control.md)
- [Lookup query language](lookup-query-language.md)
- [Free-text search query language](free-text-search-query-language.md)

# Introduction

We recommend starting with the [Examples](examples.md) to get a quick grasp of what the Documaster API is capable of.

Your mastery of the API can then be further expanded by getting acquainted with the entirety of the [Data model](model.md) and available [Endpoints](endpoints.md).

Do read through the [Access control](access-control.md) in order to gain an understanding of the behavior of our access control.

# Postman

If you're familiar with Postman you can check out our [collection of Postman samples and a description of how to get started](postman/README.md) with testing the API

# Authentication

In order to use the Documaster API a Bearer Token is required. 

Documaster is (normally) connected to Azure AD for user authentication and the Bearer Token must therefore be obtained from Azure. Obtaining a Bearer Token in Azure is described here: https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth-ropc
