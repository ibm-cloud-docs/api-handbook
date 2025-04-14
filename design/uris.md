---

copyright:
  years: 2019, 2025
lastupdated: "2025-01-16"

subcollection: api-handbook

---

{:external: .external}

# URIs
{: #uris}

## Overview
{: #overview}

URI path design is an important part of every API. Paths SHOULD be easy to read and MAY reflect the
hierarchy of underlying models. Paths SHOULD NOT end with a trailing `/`; if a client appends a
trailing `/`, the server SHOULD respond with a `301` status code along with a `Location` header
containing the correct URI.

## Version
{: #version}

The first segment of an API's path MUST be the major version of the API, prefixed with a lowercase
`v`.

## Construction
{: #construction}

Following the version segment, each path segment MUST be either a resource type or a resource
identifier. If denoting a collection of resources (e.g., `servers` in `/v2/servers`), or serving as
a prefix to an identifier (e.g., `servers` in `/v2/servers/123`), resource types MUST be plural.
Resource type names MUST be lower snake case. If a resource type segment contains uppercase
characters, the server SHOULD respond with a `404` status and appropriate error response model.

A path MUST NOT have two concurrent identifiers. Removing one or more segments from the end of a
path SHOULD yield a valid URI. This means that the existence of
`/v2/servers/123/hardware_components` implies that `/v2/servers/123`, `/v2/servers`, etc., are
valid.

## Path hierarchies
{: #path-hierarchies}

When one resource is a child of another resource, its path MAY reflect this hierarchical
relationship. However, in many cases, it may be deemed more appropriate for subordinate resources to
have their own top-level collections. Moreover, a resource's permanent location (reflected in its
`href` property and also known as its canonical location) SHOULD be under the shallowest collection
where it is represented.

Consider `/v2/users/123/tickets/456`, representing ticket `456`, belonging to user `123`. If it is
acceptable for tickets to be accessible _only_ via the users that own them, this may be a reasonable
design. But if there is ever a need for a collection of _all_ tickets, that collection would exist
at `/v2/tickets` and an individual ticket at `/v2/tickets/456`.

Additionally, if a collection of all tickets exists, it may be more practical for the collection of
tickets belonging to user `123` to exist at `/v2/tickets?user=123` than at `/v2/users/123/tickets`.
The server might also implement such collections redundantly using [non-canonical
collections](docs/api-handbook?topic=api-handbook-collections-overview#non-canonical-collection-urls).

## Constraints
{: #uri-constraints}

Services MUST have a documented and enforced service-wide length limit for URIs and this limit
SHOULD be 8000 bytes[^uri-limit-rationale]. Requests with URIs longer than the limit MUST be
rejected prior to any processing of the payload with a `414 URI Too Long` status code and
appropriate error response model.

[^uri-limit-rationale]: [RFC
   7230](https://datatracker.ietf.org/doc/html/rfc7230#section-3.1.1){: external} recommends
   supporting a _minimum_ URI length of 8000 octets. Given that various tools written to this
   standard may support _no more_ than this recommended minimum, this handbook recommends a limit of
   _exactly_ 8000 bytes.

## Path parameters
{: #path-parameters}

While not a part of HTTP per se, path parameters are the way OpenAPI represents segments in an
operation's path that contain arguments to be provided by a client at request time.

A path parameter SHOULD only be used for a resource
[identifier](/docs/api-handbook?topic=api-handbook-types#identifier), another form of
guaranteed-unique handle[^no-crn] (such as an immutable name) or a "minimally represented" resource
that can be entirely encoded as a single string (such as a tag).

[^no-crn]: But [not a CRN](https://cloud.ibm.com/docs/api-handbook?topic=api-handbook-types#crn).

A path parameter MUST NOT be used to provide a collection filter, pagination control, argument for
a [custom operation](/docs/api-handbook?topic=api-handbook-operations#custom-operations), access
token, or supplementary directive for the creation or mutation of a resource.

### Path parameter names
{: #path-parameter-names}

If a property in the response schema for an operation reflects the same value provided in the final
path parameter containing an identifier (or other unique resource handle), that property and that
parameter MUST use the same name.

For example, if the operation to retrieve a farm resource from its identifier returns:

```json
{
  "id": "0d45edaa-456f-48f2-abff-c01fd6b7530f",
  "name": "Moo-Cow Milk Farm"
}
```

Then the OpenAPI path parameter for the farm's identifier MUST be named "id":

`GET /farms/{id}`

Conversely, a path parameter MUST NOT have the same name as a top-level property in the
operation's response body that represents a different value. Additionally, a path parameter MUST
NOT have the same name as any top-level property in the operation's request body.

The path parameters used to identify a resource and any of its parent resources MUST use
consistent names across the resource type's standard operations for creating, retrieving, mutating,
deleting, and listing resources of that type.

For example, if `/farms/{farm_id}/barns/{id}` is used to retrieve, mutate, or delete a barn, the
path used to list or create barns MUST be `/farms/{farm_id}/barns` and not `/farms/{id}/barns`.

As in the example above, and where otherwise sufficient, the singular form of the prior path
segment SHOULD be used to disambiguate a parent resource's identifier from its child's identifier.
For example, `/farms/{farm_id}/barns/{barn_id}/cows/{id}` SHOULD be used instead of
`/farms/{farm_id}/barns/{farm_barn_id}/cows/{id}`.

#### Children without identifiers and custom operations
{: #unambiguous-path-parameters}

Where there is no ambiguity, path parameters SHOULD NOT be qualified. For example, if a book
resource type allows genres wholly represented by strings to be added with the path
`PUT /books/{id}/genres/{genre}` where `{genre}` is a string such as "sci-fi," and there is no
request- or response-body representation of a genre and no genre identifier, the book identifier
path parameter SHOULD remain unqualified as `{id}`.

Similarly, if a resource supports a custom operation modeled as an appended verb in a path segment,
such as `POST /servers/{id}/reboot`, the server identifier path parameter SHOULD remain unqualified
as `{id}`.

## Defining path parameters
{: #defining-path-parameters}

In an OpenAPI definition, path parameters SHOULD be defined in such a way as to maximize reuse
and minimize duplication and potential inconsistency. To this end, path parameters MUST be
enumerated in the [Path Item][path-item] object and not the [Operation][operation] object. Further,
path parameters SHOULD be defined as [Components][components] and referenced in Path Item objects.
Finally, schemas used by parameters SHOULD also be defined as separate [Components][components]
and referenced in [Parameter][parameter] objects.

[path-item]: https://spec.openapis.org/oas/v3.0.3#path-item-object
[operation]: https://spec.openapis.org/oas/v3.0.3#operation-object
[components]: https://spec.openapis.org/oas/v3.0.3#components-object
[parameter]: https://spec.openapis.org/oas/v3.0.3#parameter-object

### Example
{: #defining-path-parameters-example}

The following example demonstrates how parameters can be defined in an OpenAPI definition to
maximize reuse.

```yaml
paths:
  '/farms/{id}':
    parameters:
    - $ref: '#/components/parameters/FarmId'
    get:
      operationId: get_farm
      ...
    patch:
      operationId: update_farm
      ...
  '/farms/{farm_id}/barns':
    parameters:
    - $ref: '#/components/parameters/FarmIdQualified'
    get:
      operationId: list_farm_barns
      ...
    post:
      operationId: create_farm_barn
      ...
  '/farms/{farm_id}/barns/{id}':
    parameters:
    - $ref: '#/components/parameters/FarmIdQualified'
    - $ref: '#/components/parameters/FarmBarnId'
    get:
      operationId: get_farm_barn
      ...
    patch:
      operationId: update_farm_barn
      ...
components:
  parameters:
    FarmId:
      name: id
      in: path
      required: true
      description: The identifier for a farm
      schema:
        $ref: '#/components/schemas/Identifier'
    FarmIdQualified:
      name: farm_id
      in: path
      required: true
      description: The identifier for a farm
      schema:
        $ref: '#/components/schemas/Identifier'
    FarmBarnId:
      name: id
      in: path
      required: true
      description: The identifier for a barn on a farm
      schema:
        $ref: '#/components/schemas/Identifier'
  schemas:
    Identifier:
      type: string
      format: identifier
      pattern: '^[-0-9a-z]+$'
```

## Query parameters
{: #query-parameters}

### Constraints
{: #query-parameter-constraints}

Each query parameter supported for an operation MUST have a documented and enforced maximum length
and the sum of all query parameter maximum lengths (along with the names and control characters `&`
and `=` for each parameter) for a single operation SHOULD be less than 7000 bytes[^7000-you-say].

[^7000-you-say]: The recommended maximum URI length is 8000 bytes; leaving 1000 bytes for
   imaginatively long fully qualified domain name and path segments, it should be impossible to craft
   a URI with entirely valid parameters (and no padding) that exceeds the URI length limit. This
   allows services to reliably return user-crafted collection URIs with appended pagination tokens.

### Unrecognized and invalid parameters
{: #unrecognized-and-invalid-parameters}

To prevent [dangerous client bugs and backward-compatibility hazards][parameter-robustness],
unrecognized query parameters SHOULD and invalid parameter values MUST result in a `400` status
code and appropriate error response model.

[parameter-robustness]: /docs/api-handbook?topic=api-handbook-robustness#sanitation-and-validation

### Case insensitivity
{: #case-insensitivity}

Query parameter names SHOULD NOT be case-normalized to support case insensitivity; a parameter that
does not match the case of a defined parameter but otherwise matches its name SHOULD be treated as
any other extraneous input[^parameter-case-normalization].

However, parameter name case normalization MAY be supported for backward compatibility with
existing clients.

[^parameter-case-normalization]: Case normalization is often an error-prone process, however simple
   it may seem. One problem is that different standard libraries may not agree on the
   lowercase-equivalent value for a particular string. For example, `"İstanbul".ToLowerCase()` in
   JavaScript yields `i̇stanbul` (note the two dots over the first character), but
   `strings.ToLower("İstanbul")` in Go is more aware of locale-specific rules and yields `istanbul`.
   If the code that validates and the code that actually uses a particular parameter disagree on the
   normalization of its name, it could lead to a bug that could be exploited to validate one value
   and use another.

### Parameter duplication
{: #parameter-duplication}

Requests that provide a query string with duplicate single-value[^single-value] query parameters of
the same name and differing values[^duplicate-query-parameters] MUST result in a `400` status and
appropriate error response model. For backward compatibility with existing clients, query strings
containing duplicate query parameters of the same name and with the same value MAY be
supported[^exact-duplicate-parameters].

If a service supports parameter name [case insensitivity](#case-insensitivity), parameter names MUST
be normalized prior to validating uniqueness.

Support for array input in query parameters SHOULD use comma-separated values within a single
parameter (for example, `foo=1,2,3`) instead of duplicated[^array-parameter-duplication] parameters
(`foo=1&foo=2&foo=3`).

[^single-value]: That is, query parameters that do not support array input.

[^duplicate-query-parameters]: Silent support for ambiguously duplicated query parameters increases
   the risk that a malicious client could craft a request that bypasses critical authorization or
   validation checks.

[^exact-duplicate-parameters]: That is, exactly duplicated parameters can be treated as if only one
   parameter with the duplicated name and value was provided.

[^array-parameter-duplication]: While this kind of parameter duplication is not ambiguous per se,
   tooling support for constructing and parsing such query strings is uneven.
