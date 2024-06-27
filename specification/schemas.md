---

copyright:
  years: 2019, 2024
lastupdated: "2024-06-27"

subcollection: api-handbook

---

{:external: .note}

# Schemas
{: #schemas}

Object schemas, the expression of models in OpenAPI, generally represent a resource.

## Naming
{: #naming}

Schema names SHOULD be nouns or compound nouns represented in upper camel case. Schema names MUST
be simple, descriptive, and meaningful to developers.

### Canonical resource representation
{: #canonical-schemas}

The schema for the canonical representation of a resource in a response SHOULD correspond directly
to the path segment(s) representing the resource type and to [the way the resource type is
referenced](/docs/api-handbook?topic=api-handbook-operations) in the operation ID.

For example, a `GET /boats/{id}` operation would return a resource represented with the `Boat`
schema and a `GET /boats/{boat_id}/oars/{id}` operation would return a resource represented with
the `BoatOar` schema.

### The graph fragment pattern
{: #graph-fragment-pattern}

Non-canonical variations of a resource schema, such as those used to create or refer to a resource,
SHOULD be "graph fragment" variants of the schema. A graph fragment schema has the same structure
as its canonical schema with some properties omitted from the schema or from any nested object
schemas. For example, if a schema defines objects such as the following:

```json
{
  "name": "Stanley Q. Dallas",
  "birthdate": "1955-10-19",
  "address": {
    "street": "55 West Oak Grove",
    "city": "Happenstance",
    "state": "CT",
    "postal_code": "06058"
  }
}
```

A graph fragment variant of the same schema might define objects such as:

```json
{
  "name": "Stanley Q. Dallas",
  "address": {
    "postal_code": "06058"
  }
}
```

### Resource collection schemas
{: #collection-schemas}

The schema for a [resource collection](/docs/api-handbook?topic=api-handbook-collections-overview)
returned by a [list operation](/docs/api-handbook?topic=api-handbook-operations#standard-operations)
SHOULD be the name of the canonical schema for the resource type, suffixed with `Collection`.

For example, a `GET /goat/{goat_id}/chores` operation would return a resource collection
represented with the `GoatChoreCollection` schema.

Each resource in a collection SHOULD be represented with its canonical schema. If performance
considerations make it infeasible to return the canonical representation of a schema, a
lighter-weight schema variant suffixed with `Summary` MAY be used. The summary representation of a
resource SHOULD include all the critical information that would be expected in a user interface or
command-line interface listing such resources.

For example, a `GreebleCollection` would contain an array of resources represented with the
`Greeble` schema or, if infeasible, a `GreebleSummary` schema.

A summary schema MUST be a [graph fragment](#graph-fragment-pattern) variant of the canonical
schema for the resource type.

### Resource creation and replacement schemas
{: #creation-and-replacement-schemas}

The request schema used in an operation to create a new resource with a `POST` (or `PUT`) request or
to replace a resource[^schema-for-replacement] with a `PUT` request SHOULD be the name of the
canonical schema for the resource type, suffixed with `Prototype`.

[^schema-for-replacement]: The requirement that a resource creation and replacement use the same
   schema is a deliberate constraint on what it means to "replace" a resource. In particular,
   property defaults defined in a schema MUST be used in a replacement context, rather than re-using
   prior values when replacing an existing resource.

For example, a `POST /bulldozers` operation would accept the information needed to create a new
bulldozer represented with the `BulldozerPrototype` schema.

A prototype schema MUST be a [graph fragment](#graph-fragment-pattern) variant of the canonical
schema except where a property is required by a domain-specific constraint to be explicitly
write-only.

There is an important exception to this guidance if the nature of an API is to provide clients with
full control over entire resources, with no system-defined or immutable aspects. For such APIs,
the canonical schema itself SHOULD be used in requests to create or replace a resource.
{: note}

### Resource reference schemas
{: #reference-schemas}

When a related resource is referenced within another resource's schema, properties of the related
resource SHOULD be represented with a schema with the name of the related resource's canonical
schema, suffixed with `Reference`. The property that a resource reference is embedded in MAY be
named for the related resource type or (especially if the relationship is polymorphic) the nature
of the relationship.

A resource reference MUST be a [graph fragment](#graph-fragment-pattern) variant of the canonical
schema (and, if it exists, the summary schema) for the resource type except for properties that
indicate a resource no longer exists.

The resource's unique, immutable handle (usually its `id`) as represented in the final path
parameter of its canonical URI MUST be included in its reference. A human-friendly handle for the
resource, such as a user-defined name SHOULD also be included in the reference. If the referenced
resource is native to the service in which the reference is returned, its `href` SHOULD be included.
Finally, if the resource has a CRN, its `crn` property SHOULD also be included. Other properties
SHOULD NOT be included in a resource reference.

### Resource identity schemas
{: #identity-schemas}

When a related resource is referenced within a request schema (such as a resource prototype), a
schema with the name of the related resource's canonical schema, suffixed with `Identity`, should
be used for the same property name used to reference the resource in the response.

For resources native to the service in which the identity is needed, the identifying property MUST
be the unique, immutable handle (usually its `id`) as represented in the final path parameter of
the resource's canonical URI and within the canonical schema. For resources external to the
service, the `crn` property SHOULD be used.

### Resource patch schemas
{: #patch-schemas}

The request schema used in an operation to update a resource with a [JSON merge
patch][rfc7396]{: external} SHOULD be the name of the canonical schema for the resource type,
suffixed with `Patch`. In accordance with the JSON merge patch format, this schema MUST be a [graph
fragment](#graph-fragment-pattern) of the canonical schema, and MUST NOT have any required
properties.

[rfc7396]: https://datatracker.ietf.org/doc/html/rfc7396

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
