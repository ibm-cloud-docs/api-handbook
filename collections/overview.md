---

copyright:
  years: 2019, 2025
lastupdated: "2025-01-16"

subcollection: api-handbook

---

{:external: .external}

# Collections overview
{: #collections-overview}

This section details how collections should behave. Collections SHOULD have a URL ending in a plural
resource name, such as `/v2/accounts` or `/v1/farms/{farm_id}/barns`.

## Response format
{: #response-format}

The representation of a collection MUST be an object containing a field with the same plural
resource name appearing in the collection's URL. This field MUST contain an array of resources in
the collection. This response object MAY also include collection metadata, such as for
[pagination](/docs/api-handbook?topic=api-handbook-pagination).[^collection-response]

### Example collection response
{: #example-collection-response}

```json
{
  "accounts": [
    {
      "id": "499aed3c-3f49-4a04-8e69-44c2f2894195",
      "company_name": "Aperture Science",
      "href": "https://api.example.com/v2/accounts/499aed3c-3f49-4a04-8e69-44c2f2894195"
    },
    {
      "id": "2f27ff7b-3183-4e9a-a085-db457402ee95",
      "company_name": "Black Mesa",
      "href": "https://api.example.com/v2/accounts/2f27ff7b-3183-4e9a-a085-db457402ee95"
    }
  ]
}
```

## Non-canonical collection URLs
{: #non-canonical-collection-urls}

A resource has only one _canonical_ location (also known as its _permanent_ location), which is
reflected in its `href` property. By extension, a resource has only one canonical collection, such
as `/v2/accounts` in the example above. However, relationships with other resources may warrant
adding _non-canonical_ collections. Consider a [conference speaker
scenario](/docs/api-handbook?topic=api-handbook-operations#bindings-to-multiple-other-resources),
where `PUT /v1/conferences/{conference_id}/speakers/{id}` is used to add a speaker to a conference.
In this case, `/v1/conferences` and `/v1/speakers` are the canonical collections for conferences and
speakers respectively. In contrast, `/v1/conferences/{conference_id}/speakers` is the non-canonical
collection listing the speakers for a given conference.

Since non-canonical collections are intended to list associations between resource types, and access
to those resource types may require different authorizatons, the representation of the collection
SHOULD return [reference schemas](/docs/api-handbook?topic=api-handbook-schemas#reference-schemas) rather than canonical or summary schemas.

### Example non-canonical collection response
{: #example-non-canonical-collection-response}

```json
{
  "speakers": [
    {
      "id": "1ed3d170-5cd7-466e-8c11-787d0228809f",
      "name": "brian-kernighan",
      "href": "https://api.example.com/v1/speakers/1ed3d170-5cd7-466e-8c11-787d0228809f"
    },
    {
      "id": "fa25c029-d7b9-4348-805b-8bd3b5eed8f2",
      "name": "vint-cerf",
      "href": "https://api.example.com/v1/speakers/fa25c029-d7b9-4348-805b-8bd3b5eed8f2"
    }
  ]
}
```

The shown `href` values reflect the speaker's canonical location, which a client can use to
retrieve the speaker's canonical schema (if the client has permission).
{:note: .note}

## Wildcard collection URLs
{: #wildcard-collection-urls}

To facilitate reading resources across collections, one or more path parameters in a collection's
request URL MAY be wildcarded with the `-` (hyphen or dash) character. For example, `GET
/v1/farms/-/barns` requests all barns, regardless of the farm.

Wildcards MUST be limited to collection `GET` methods, which may only support wildcards for path
parameters. Path parameters that support wildcards MUST be explicitly documented. Wildcard support
MUST NOT affect any pagination, filtering, and sorting support otherwise provided by a collection
method. The sorting requirement means wildcards MUST NOT be used if duplicate resources could be
returned, or if not all sort options can be supported.

## Individual resource URLs
{: #individual-resource-urls}

Resources belonging to a collection MAY be individually addressable, and if so, their URLs SHOULD
start with the URL of the collection.[^hierarchical-url]  For example, an account in the
`/v2/accounts` collection would have the URL `/v2/accounts/:account_id` where `:account_id` is the
unique account identifier.

Each individual resource's canonical URL SHOULD be included in the root of its representation as the
`href` property. This URL SHOULD be complete and begin with the protocol (e.g.,
`https://api.example.com/v2/accounts/499aed3c-3f49-4a04-8e69-44c2f2894195`). The wildcard character
(`-`) MUST NOT appear in the property value (i.e., canonical parent identifiers MUST be used).

## Wildcard individual URLs
{: #wildcard-individual-urls}

As a convenience for clients, resource URLs that include parent resource identifiers but are
uniquely identified[^uniquely-identified] by the final identifier MAY allow clients to wildcard any
parent resource identifiers with the `-` (hyphen or dash) character (e.g. `GET
/v1/farms/-/barns/dc0eff9d-5a8c-46b9-9553-e1a0e554caaf`). The final identifier MUST NOT be
wildcarded, and any path parameters that support wildcards MUST be explicitly documented. The server
MUST NOT return the resource, but instead return [`301 Moved` with a `resolved`
code](/docs/api-handbook?topic=api-handbook-status-codes#301-302-303-and-307-response-bodies)

[^uniquely-identified]: That is, the resource identifier is unique by virtue of an architectural guarantee, not transiently unique because no ambiguity currently exists in the system's state.

[^collection-response]: Encapsulation of arrays is always required for reasons outlined under the
   [Object
   Encapsulation](/docs/api-handbook?topic=api-handbook-format#object-encapsulation)
   section of [Format](/docs/api-handbook?topic=api-handbook-format).

[^hierarchical-url]: RFC 3986 [defines URLs as
   hierarchical](https://datatracker.ietf.org/doc/html/rfc3986#section-1.2.3){: external} and the
   `/` character is used to delineate segments of the hierarchy.
