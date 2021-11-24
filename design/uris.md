---

copyright:
  years: 2019, 2021
lastupdated: "2021-06-30"

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
have their own top-level collections. Moreover, a resource's permanent location (represented in its
`href` property) SHOULD be under the shallowest collection where it is represented.

Consider the following path representing ticket `456`, belonging to user `123`:
`/v2/users/123/tickets/456`. If it is acceptable for tickets to be accessible _only_ via the users
that own them, this may be a reasonable design. But if there is ever a need for a collection of
_all_ tickets, a collection would exist at, for example, `/v2/tickets` and an individual ticket at
`/v2/tickets/456`.

Additionally, if a collection of all tickets exists, it may be more practical for the collection of
tickets belonging to user `123` to exist at `/v2/tickets?user=123` than at `/v2/users/123/tickets`.
It is not forbidden, however, for a server to implement such collections redundantly.

## Contraints
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

## Query parameters
{: #query-parameters}

### Contraints
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
