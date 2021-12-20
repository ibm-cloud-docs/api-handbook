---

copyright:
  years: 2019
lastupdated: "2019-12-01"

subcollection: api-handbook

---

{:external: .external}

# Collections overview
{: #collections-overview}

This section details how collections should behave. Collections SHOULD have a URL ending in a plural
resource name, such as `/v2/accounts`.

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
      "href": "https://api.example.com/v1/accounts/499aed3c-3f49-4a04-8e69-44c2f2894195"
    },
    {
      "id": "2f27ff7b-3183-4e9a-a085-db457402ee95",
      "company_name": "Black Mesa",
      "href": "https://api.example.com/v1/accounts/2f27ff7b-3183-4e9a-a085-db457402ee95"
    }
  ]
}
```

## Individual resource URLs
{: #individual-resource-urls}

Resources belonging to a collection MAY be individually addressable, and if so, their URLs SHOULD
start with the URL of the collection.[^hierarchical-url]  For example, an account in the
`/v2/accounts` collection would have the URL `/v2/accounts/:account_id` where `:account_id` is the
unique account identifier.

Each individual resource's URL SHOULD be included in the root of its representation as the `href`
property. This URL SHOULD be complete and begin with the protocol (e.g.,
`https://api.example.com/v2/accounts/499aed3c-3f49-4a04-8e69-44c2f2894195`).

[^collection-response]: Encapsulation of arrays is always required for reasons outlined under the
   [Object
   Encapsulation](/docs/api-handbook/collections?topic=api-handbook-format#object-encapsulation)
   section of [Format](/docs/api-handbook/collections?topic=api-handbook-format).

[^hierarchical-url]: RFC 3986 [defines URLs as
   hierarchical](https://datatracker.ietf.org/doc/html/rfc3986#section-1.2.3){: external} and the
   `/` character is used to delineate segments of the hierarchy.
