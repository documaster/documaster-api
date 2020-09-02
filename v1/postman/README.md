# Introduction

The `Documaster Integration API.postman_collection.json` file contains a v2.1 collection that can be imported into Postman.

It contains close to 100 sample requests exhibiting the functionality of the following endpoints for each of the publicly-exposed resources:
- Fetch endpoint
- Fetch by ID endpoint
- Fetch related endpoint
- Create endpoint
- Update endpoint
- Delete endpoint
- Lookup endpoint 
- Upload endpoint
- Download endpoint

# Postman environment variables

Two variables have been used throughout the Postman requests:
- token
- documaster-url

`documaster-url` must point to the Documaster instance including port information. For example: https://documaster.com:443.

`token` must contain a valid Bearer token to be used for authorization purposes.

These variables must be configured by creating a local Postman environment.
