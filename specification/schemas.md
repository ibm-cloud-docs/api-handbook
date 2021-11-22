---

copyright:
  years: 2019
lastupdated: "2019-12-01"

subcollection: api-handbook

---

# Schemas
{: #schemas}

Object schemas, the expression of models in OpenAPI, generally represent a resource.

## Naming
{: #naming}

Schema names SHOULD be nouns. Schema names MUST be simple, descriptive, and meaningful to
developers.

## Descriptions
{: #descriptions}

Every schema and property SHOULD have a description. Schema descriptions SHOULD NOT include
references to implementation details, such as describing a schema as a “JSON object”. Rather, use
the generic “object” in schema descriptions.

## Nested object schemas
{: #nested-object-schemas}

All object schemas MUST be defined separately in the `components / schemas` section of the API
document, and MUST NOT be nested within the definition of another schema.[^no-nested-schemas]

[^no-nested-schemas]: A nested schema definition creates an "anonymous" schema which may have
   an arbitrary name assigned to it in implementations.
