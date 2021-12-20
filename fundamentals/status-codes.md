---

copyright:
  years: 2019
lastupdated: "2019-12-01"

subcollection: api-handbook

---

{:external: .external}

# Status codes
{: #status-codes}

HTTP has a set of standard status codes. These fall into five categories:

*  Informational: `1xx`
*  Success: `2xx`
*  Redirection: `3xx`
*  Client Errors: `4xx`
*  Server Errors: `5xx`

## Informational: 1xx
{: #informational-1xx}

| Code  | Meaning | Description |
| ----- | ------- | ----------- |
| `100` | [Continue](https://datatracker.ietf.org/doc/html/rfc7231#section-6.2.1){: external} | The server MAY respond with this status code if the client sends a preliminary request (sometimes called a "look-before-you-leap" request); this code indicates the client should resend the request, in full (including the previously omitted representation). This code MUST NOT be returned unless the `Expect` request header was provided with a value of `100-continue`. |
| `101` | [Switching Protocols](https://datatracker.ietf.org/doc/html/rfc7231#section-6.2.2){: external} |This status MUST be returned for successful requests that include a supported use of the `Upgrade` header. This status MUST NOT be returned except for operations which exist specifically to switch protocols[^only-for-protocol-switching]. For example, the HTTP handshake for a WebSocket connection returns `101 Switching Protocols` and this handshake MAY be documented as an operation in an HTTP API. |
{: caption="Informational status codes" caption-side="bottom"}

[^only-for-protocol-switching]: That is, services MUST NOT expect or encourage clients to handle protocol switching as a part of operations which may result in normal, successful (`2xx`) HTTP responses.

## Success: 2xx
{: #success-2xx}

| Code  | Meaning | Description |
| ----- | ------- | ----------- |
| `200` | [OK](https://datatracker.ietf.org/doc/html/rfc7231#section-6.3.1){: external} | This code MUST be returned for successful requests not covered by a more specific `2xx` code. The response body to return depends on the method; see [Methods](/docs/api-handbook?topic=api-handbook-methods) for more details. |
| `201` | [Created](https://datatracker.ietf.org/doc/html/rfc7231#section-6.3.2){: external} | This code MUST be returned when a new resource was successfully created in a synchronous manner. When returning this code, the server MUST also return a `Location` header with the URI of the created resource. Depending on the `Prefer` header, a representation of the resource MAY be included in the response body; see the [POST section](/docs/api-handbook?topic=api-handbook-methods#post) for more details. |
| `202` | [Accepted](https://datatracker.ietf.org/doc/html/rfc7231#section-6.3.3){: external} | This code MUST be returned when the request will be processed asynchronously, and may be rejected when it's actually processed. A resource representing the status of the request SHOULD be included in the response, and the response `Location` header SHOULD include a URI where this same status may be retrieved so that the client may stay up-to-date. |
| `204` | [No Content](https://datatracker.ietf.org/doc/html/rfc7231#section-6.3.5){: external} | This code MUST be returned when the server successfully processed the request and is not returning any content. This MAY be used for `DELETE` requests or requests where the client includes a `Prefer` header containing `return=minimal`. It MAY occasionally be used for `PUT` or `POST` requests where it is impractical to return a resource representation because of size or cost, or for `GET` requests where a resource exists but has an empty representation. |
| `206` | [Partial Content](https://datatracker.ietf.org/doc/html/rfc7233#section-4.1){: external} | This code indicates that the server is delivering only part of the resource. Servers MUST use this when enabling the client to resume interrupted downloads, or to split a download into multiple simultaneous streams.[^content-range] |
{: caption="Success status codes" caption-side="bottom"}

### Requests requiring no action
{: #requests-requiring-no-action}

If a request attempts to put a resource into a state which it is already in, the status code SHOULD
be in the `2xx` range. In other words, the status code SHOULD match what would be given if a change
had been required.

This may not always be practical, however. If a client makes a `DELETE` request for a resource that
has already been deleted, the server is not required to maintain a record of the deleted resource
and MAY respond with `404`.[^already-deleted-resource]

## Redirection: 3xx
{: #redirection-3xx}

| Code  | Meaning | Description |
| ----- | ------- | ----------- |
| `300` | [Multiple Choices](https://datatracker.ietf.org/doc/html/rfc7231#section-6.4.1){: external} | This code MAY be returned in the uncommon scenarios where the URI requested does not unambiguously specify a particular resource or representation of a resource. See [URI Ambiguity](#uri-ambiguity) below. |
| `301` | [Moved Permanently](https://datatracker.ietf.org/doc/html/rfc7231#section-6.4.2){: external} | This code SHOULD be returned when a resource's URI changes, in order to keep the previous URI from failing. When this code is returned, the server MUST also include a `Location` header. It indicates that the current request, and all future requests, should be directed to the URI provided in the `Location` header. |
| `302` | [Found](https://datatracker.ietf.org/doc/html/rfc7231#section-6.4.3){: external} | This code SHOULD NOT be used. Instead, temporary redirects should respond with `303` or `307`, as applicable, to avoid ambiguity.[^302-ambiguity] It MAY only be used if there is strong evidence that clients of a service will not be able to parse or understand status codes `303` and `307`. |
| `303` | [See Other](https://datatracker.ietf.org/doc/html/rfc7231#section-6.4.4){: external} | This code MAY be returned to indicate the request was processed and more information can be obtained with a `GET` request to the URI provided in the `Location` header, which MUST be included when this code is returned. This is most useful in such cases where a resource at another location is relevant to a request made with a `POST`, `PUT`, or `DELETE` request. |
| `304` | [Not Modified](https://datatracker.ietf.org/doc/html/rfc7232#section-4.1){: external} | This code MAY be returned to indicate that the previously-downloaded resource a client possesses is up-to-date, and there's no need to resend. This code MUST NOT be returned except in response to conditional requests. Responses with a `304` status MUST NOT have a response body. |
| `307` | [Temporary Redirect](https://datatracker.ietf.org/doc/html/rfc7231#section-6.4.7){: external} | This code MAY be returned to indicate that the request should be resubmitted to the URI provided in the `Location` header, which MUST be included when this code is returned, and that future requests should use the original URI. This code indicates that the request method should not change when using the temporary URI. |
{: caption="Redirection status codes" caption-side="bottom"}

### URI ambiguity
{: #uri-ambiguity}

URI ambiguity requiring the use of a `300` status code SHOULD be avoided by following the below
principles:

1. URI schemes SHOULD be inherently unambiguous with respect to what resource is meant.
2. If multiple representations of a particular resource are offered, the desired version SHOULD be
   specified with an `Accept` header rather than distinct URIs, and a default representation SHOULD
   be served liberally.[^300-accept]

However, in such cases where it is deemed important that an occasionally-ambiguous URI scheme be
used or where different representations of a resource require differing URIs (e.g., for media files
where a file extension is considered significant), a `300` status code MAY be returned where a
particular URI is ambiguous.

### 303 and 307 temporary redirects
{: #303-and-307-temporary-redirects}

A `303` in response to a `POST`, `PUT`, or `DELETE` indicates that the operation has succeeded but
that the response entity-body is not being sent along with this request. If the client wants the
response entity-body, it needs to make a `GET` request to the URI provided in the `Location` header.

A `307` in response to a `POST`, `PUT`, or `DELETE` indicates that the server has not even tried to
perform the operation and the client needs to resubmit the entire request, with the original method,
to the URI in the `Location` header.

## Client errors: 4xx
{: #client-errors-4xx}

 | Code  | Meaning | Description |
 | ----- | ------- | ----------- |
 | `400` | [Bad Request](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.1){: external} | This code MUST be returned when a request cannot be processed due to client error (e.g., the request is malformed or too large) when a more specific code is not applicable. |
 | `401` | [Unauthorized](https://datatracker.ietf.org/doc/html/rfc7235#section-3.1){: external} | This code MUST be returned when a request needs to be authenticated to succeed or the credentials provided are invalid. Responses with a `401` status code MUST also provide a valid `WWW-Authenticate` header.[^401-403-meaning] |
 | `403` | [Forbidden](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.3){: external} | This code SHOULD be returned when the client is not allowed to make the desired request. This status SHOULD be returned if the client is properly authenticated but does not possess permission to perform the request, or if the client is not authenticated, but authentication would not affect the outcome. |
 | `404` | [Not Found](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.4){: external} | This code SHOULD be returned when the requested resource could not be found but may be available in the future. This status code MAY also be used for security reasons in the scenario [described below](#security-implications-of-401-and-403). |
 | `405` | [Method Not Allowed](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.5){: external} | This code SHOULD be returned when the requested resource doesn't support the request method. When this code is returned the response MUST include the `Allow` header with the list of accepted request methods for the resource. This code MUST only be used when one of the [Methods](/docs/api-handbook?topic=api-handbook-methods) documented in this handbook is not valid for a particular URI. See also `501` under [server errors](#server-errors-5xx).[^405-versus-501] |
 | `406` | [Not Acceptable](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.6){: external} | This code MUST be returned when the resource the client requested is not available in a format allowed by the `Accept` header the client supplied. |
 | `408` | [Request Timeout](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.7){: external} | This code SHOULD be returned when the client took too long to finish sending the request. |
 | `409` | [Conflict](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.8){: external} | This code SHOULD be returned when the request cannot be processed because of a conflict between the request and the current client-controlled state in the system. See [Conflicts](#conflicts). |
 | `410` | [Gone](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.9){: external} | This code MAY be returned to indicate that a resource is no longer available and is not expected to return to availability. However, if it is impractical for a system to maintain a record of deleted or purged resources, or poses a security risk, `404` MAY be returned instead. |
 | `412` | [Precondition Failed](https://datatracker.ietf.org/doc/html/rfc7232#section-4.2){: external} | This code MUST be returned when the client specified one or more preconditions in its headers, and the server cannot meet those preconditions. |
 | `413` | [Payload Too Large](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.11){: external} | This code MUST be returned when a request payload size exceeds [the documented limit](/docs/api-handbook?topic=api-handbook-format#request-body-constraints). |
 | `414` | [URI Too Long](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.12){: external} | This code MUST be returned when a request's URI length exceeds the [the documented limit](/docs/api-handbook?topic=api-handbook-uris#constraints) |
 | `415` | [Unsupported Media Type](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.13){: external} | This code MUST be returned when the client sends a payload of a content type the server cannot accept.[^content-type-versus-invalid-data] |
 | `422` | [Unprocessable Entity](https://datatracker.ietf.org/doc/html/rfc4918#section-11.2){: external} | This code SHOULD NOT be used. Instead, `400` should be returned in response to invalid request payloads.[^422-use] |
 | `429` | [Too Many Requests](https://datatracker.ietf.org/doc/html/rfc6585#section-4){: external} | This code MUST be returned when rate-limiting is in use, and the client has sent too many requests for a given time period. |
{: caption="Client error status codes" caption-side="bottom"}

### Security implications of 401 and 403
{: #information-exposure-with-401-and-403}

There may be times when returning a `401` or a `403` would reveal the presence of a resource on the
server that a client should not be allowed to know exists. In these cases, the server MUST
respond with the status code that would be used if the resource did not exist, even for requests
that would succeed with proper authentication.

For example, for a `GET` request for a resource that the client is not allowed to know exists, the
server MUST respond with a `404 Not Found`. Or, for the creation of a binding to that resource,
the server MUST respond with a `400 Bad Request`, exactly as if the resource did not exist.

### Conflicts
{: #conflicts}

Though [RFC 7231][rfc-7231-409] suggests `409 Conflict` be limited to conflicts between a request
and the "target resource," this handbook recommends a broader view of what constitutes a
conflict. Specifically, a request SHOULD be deemed in conflict (and receive a `409` status code) if
the request is syntactically and semantically valid but cannot be processed because of the
client-mutable state of one or more existing resources that would be used by the request in a way
the system deems invalid.

`409 Conflict` SHOULD NOT be returned for a request that the system can determine could not be
successfully resubmitted after the client made other requests to resolve conflicts with the state
of existing resources.

[rfc-7231-409]: https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.8

The following scenarios SHOULD result in a `409 Conflict`:

*  A request would update a certain property in a resource with a value that isn't compatible with
   the current value of another property in the resource.
*  A request would update or create a resource with a binding to an existing resource that already
   has an exclusive binding or has reached its binding limit.
*  A request would use an existing resource (for example, to generate a new resource) which has
   client-mutable state preventing such use.
*  A request would delete a resource that another resource depends on.

The following scenarios MUST NOT result in a `409 Conflict`:

*  A request has an internal conflict. This SHOULD result in a `400 Bad Request`.
*  A request depends on a resource that does not exist. This SHOULD result in a `400 Bad Request`.
*  A request would update or create a resource with a binding to an existing resource that the user
   is unauthorized to bind to. This SHOULD result in a `403 Forbidden` unless the user is [not
   permitted to know about](#information-exposure-with-401-and-403) the existing resource.
*  A request would conflict with state that is invisible to clients. This MUST NOT happen by design,
   but unexpected runtime scenarios MAY result in a `500 Internal Server Error`.
*  A request would conflict with state that is immutable for clients. This SHOULD result in a `400
   Bad Request`.
*  A safe[^safe-requests] request cannot be made because of the current state of the system. This
   SHOULD NOT happen by design. Consider modeling the missing functionality as an "empty" resource
   with a `200 OK` response or an absent resource with a `404 Not Found` response.

The following scenarios SHOULD NOT result in a `409 Conflict`:

*  A request cannot be processed because of _transient_ resource state and the client is advised to
   wait for that state to change and retry the request. This SHOULD NOT happen by design, but if it
   does, it MAY result in a `409 Conflict`.
*  A request would update or create a resource with a binding to an existing resource which has
   irrecoverably failed or expired. This SHOULD result in a `400 Bad Request`.
*  A request would use an existing resource (for example, to generate a new resource) which has
   irrecoverably failed or expired. This SHOULD result in a `400 Bad Request`.

[^safe-requests]: Safe requests are generally limited to those with `GET` or `HEAD` methods but may
   also include a safe `POST` request in [certain scenarios][post-other-uses].

[post-other-uses]: /docs/api-handbook?topic=api-handbook-methods#post-other-uses

## Server errors: 5xx
{: #server-errors-5xx}

| Code  | Meaning | Description |
| ----- | ------- | ----------- |
| `500` | [Internal Server Error](https://datatracker.ietf.org/doc/html/rfc7231#section-6.6.1){: external} | This code MUST be returned when a fatal error caused by an unexpected condition occurs on the server and was not caused by the client.[^500-detectable-errors] |
| `501` | [Not Implemented](https://datatracker.ietf.org/doc/html/rfc7231#section-6.6.2){: external} | The code MUST be returned when the server does not recognize the request method. This code MUST only be used for methods not documented under [Methods](/docs/api-handbook?topic=api-handbook-methods). See also `405` under [client errors](#client-errors-4xx). |
| `503` | [Service Unavailable](https://datatracker.ietf.org/doc/html/rfc7231#section-6.6.4){: external} | This code SHOULD be returned when the server is temporarily unavailable because it is overloaded or down for maintenance. The server MAY include a `Retry-After` header telling the client when it should try submitting the request again. |
| `505` | [HTTP Version Not Supported](https://datatracker.ietf.org/doc/html/rfc7231#section-6.6.6){: external} | This code SHOULD be returned when the server does not support the HTTP protocol version used in a request. The response SHOULD contain a document describing which protocols the server does support. |
{: caption="Server error status codes" caption-side="bottom"}

### 500 and 503 errors
{: #500-and-503-errors}

Because the client can do nothing to fix a `500` or `503` status code (and can only try again
later), it MUST be considered a critical application failure whenever one of those status codes is
returned in production. For more information on this and error response models, see the handbook's
documentation on [Errors](/docs/api-handbook?topic=api-handbook-errors).


[^content-range]: If the entity-body is a single byte range from the representation, the response as
   a whole must have a `Content-Range` header explaining which bytes of the representation are being
   served. If the body is a multipart entity (that is, multiple byte ranges of the representation
   are being served), each part must have its own `Content-Range` header.

[^already-deleted-resource]: Note that this doesn't prevent `DELETE` from being idempotent because
   repeated `DELETE` requests still result in the same _state_ even if the responses differ.

[^302-ambiguity]: Historically, HTTP clients have not honored the original standard for `302`
   behavior. Because of the differences between the specification and commonly-expected behavior,
   `302` should be avoided entirely in favor of `303` and `307`. RFC 2616 provides [explicit
   insight](https://datatracker.ietf.org/doc/html/rfc2616#section-10.3.3){: external} on this
   matter.

[^300-accept]: A `406` status code should be returned for any `Accept` header value that cannot be
   fulfilled.

[^401-403-meaning]: Details on the semantics of `401` and `403` can be found in a [2010 working
   group email](https://lists.w3.org/Archives/Public/ietf-http-wg/2010JulSep/0085.html){: external}
   from Roy T. Fielding.

[^405-versus-501]: `405` is similar to `501` (“Not Implemented”), but `501` should only be used when
   the server doesn’t recognize the method at all. In contrast, `405` should be returned when the
   client uses a recognized method that isn't supported by the URI.

[^content-type-versus-invalid-data]: Note that a `400` should be returned if the content type is
   acceptable but the payload is otherwise invalid.

[^422-use]: The `422` status was at one time favored for requests containing payloads with valid   
   syntax but invalid semantics because RFC 2616 [required   
   that](https://datatracker.ietf.org/doc/html/rfc2616#section-10.4.1){: external} `400` only be
   used "due to malformed syntax." However, RFC 7231, which obsoletes RFC 2616, [makes no such
   stipulation](https://datatracker.ietf.org/doc/html/rfc7231#section-6.5.1){: external}. Because
   `400` is no longer disallowed and because `422` is defined only in an [RFC targeting
   WebDAV](https://datatracker.ietf.org/doc/html/rfc4918){: external}, `400` is considered the
   better choice.

[^500-detectable-errors]: This error can also be raised deliberately in the case of a detected but
   unrecoverable error such as a failure to communicate with another service component. In all
   cases, however, `500` errors should be [treated as critical
   failures](/docs/api-handbook?topic=api-handbook-errors).
