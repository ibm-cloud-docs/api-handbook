---

copyright:
  years: 2021
lastupdated: "2021-05-05"

subcollection: api-handbook

---

# Resources
{: #resources}

The design of resources is fundamental to a usable API. That is, APIs should be designed primarily
around how the state of a service is modeled as resources and secondarily around operations
available to manipulate those resources.

This distinction allows users of the APIs to learn and reason about a resource's state separately
from how that state can be changed. It's also a familiar separation of concepts for programmers:
any programming environment has some notion of structured data (from low-level constructs, such as
blocks of memory, to high-level constructs, such as classes) and code that operates on it (from
subroutines in assembly to methods on objects).

By defining the data structures — or _resources_ — first, an API is more approachable. The REST
architectural style for HTTP APIs is built on this idea. 

Further, HTTP provides a standard way to represent a resource's canonical address in the form of a
URL, and standard methods to create (with `POST`), list and read (with `GET`), mutate (with `PUT`
and `PATCH`) and delete (with `DELETE`) resources. With standard methods available for each
resource, users can apply knowledge and code across operations that each work in a predictable,
robust way.

In IBM Cloud REST-style APIs, each resource type SHOULD have two standard URL path types:

1. A resource collection path, such as `/keys`. A `GET` request on the collection returns a list of
   resources within it. A `POST` request on a collection URL creates a new resource in the
   collection.
2. A resource path, such as `/keys/{id}` where `{id}` is a unique identifier for a specific
   resource. A `GET` request on a resource URL returns the canonical resource representation,
   `PATCH` updates a resource, `PUT` replaces a resource, and `DELETE` deletes a resource.

The schemas for request payloads to create and mutate resources SHOULD be consistent with the
canonical representation of the resource returned by a `GET` on the resource URL and MUST NOT differ
arbitrarily. Often, schemas for resource creation and mutation MAY represent a subset of the
properties in a resource's canonical representation, omitting properties that are system-defined or
immutable.

For example, consider a resource's canonical representation:

```json
GET /keys/b34f2d29-25ff-40ac-8762-78bad7e3eea9
---
{
  "id": "b34f2d29-25ff-40ac-8762-78bad7e3eea9",
  "href": "https://service.example.com/keys/b34f2d29-25ff-40ac-8762-78bad7e3eea9",
  "name": "my-key",
  "value": "VGhlcmUgd2FzIG5vIHBvc3NpYmlsaXR5IG9mIHRha2luZyBhIHdhbGsgdGhhdCBkYXkuIFdlIGhhZCBiZWVuIHdhbmRlcmluZywgaW5kZWVkLCBpbiB0aGUgbGVhZmxlc3Mgc2hydWJiZXJ5IGFuIGhvdXIgaW4gdGhlIG1vcm5pbmc7IGJ1dCBzaW5jZSBkaW5uZXIgKE1ycy4gUmVlZCwgd2hlbiB0aGVyZSB3YXMgbm8gY29tcGFueSwgZGluZWQgZWFybHkpIHRoZSBjb2xkIHdpbnRlciB3aW5kIGhhZCBicm91Z2h0IHdpdGggaXQgY2xvdWRzIHNvIHNvbWJyZSwgYW5kIGEgcmFpbiBzbyBwZW5ldHJhdGluZywgdGhhdCBmdXJ0aGVyIG91dGRvb3IgZXhlcmNpc2Ugd2FzIG5vdyBvdXQgb2YgdGhlIHF1ZXN0aW9uLg=="
  "zone": "us-south-1"
}
```

In such a resource type, `id` and `href` might be considered to be system-defined and thus omitted
from the request schemas for `POST /keys` and `PATCH /keys/{id}`. Further, the key's `value` and
`zone` could be considered immutable, and thus be included in the request schema for `POST /keys`
but omitted from the request schema for `PATCH /keys/{id}`.

Properties that are included in such schemas MUST be defined consistently with a resource's
canonical representation. For example, a `PATCH` request to change the `name` for the key in the
previous example would look like:

```json
PATCH /keys/b34f2d29-25ff-40ac-8762-78bad7e3eea9
{
  "name": "my-renamed-key"
}
```

... and MUST NOT look like:

```json
PATCH /keys/b34f2d29-25ff-40ac-8762-78bad7e3eea9
{
  "label": "my-renamed-key"
}
```

Responses to requests that create or mutate a resource SHOULD return the resource's canonical
representation. Rarely, a resource MAY have a "write-only" property represented in a request schema
but not in its response schema.
