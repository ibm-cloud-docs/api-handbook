---

copyright:
  years: 2019, 2025
lastupdated: "2025-04-10"

subcollection: api-handbook

---

{:external: .external}

# Headers
{: #headers}

This section details the header usage, particularly where there are guidelines additional to the
HTTP specification. Most of the headers detailed below are not required to be supported in most
scenarios, but when any of the below headers are used, they MUST follow the constraints within the
HTTP specification and within this section of the handbook.

## Representation headers
{: #representation-headers}

| Header | Type | Description |
| ------ | ---- | ----------- |
| [Content-Encoding](https://datatracker.ietf.org/doc/html/rfc7231#section-3.1.2.2){: external} | Request, Response | If a response body is encoded, this header MUST reflect the encoding used. Otherwise, this header MUST be omitted from the response. Requests including this header with an encoding not supported by the server MUST be rejected with a `415 Unsupported Media Type` status code and appropriate error response model. See [below](#content-encoding) for more details. |
| [Content-Language](https://datatracker.ietf.org/doc/html/rfc7231#section-3.1.3.2){: external} | Request, Response | If a response body is intended for an audience who speak a particular language (or languages), this header SHOULD be used to indicate that language (or those languages). If this header is included in a request it MAY be used by the server to classify the language of the incoming content appropriately. |
| [Content-Location](https://datatracker.ietf.org/doc/html/rfc7231#section-3.1.4.2){: external} | Request, Response | When applicable, the `Content-Location` response header SHOULD be used to indicate the permanent location of the returned resource. See [below](#content-location) for more details. For requests, the `Content-Location` MUST be ignored by the server except for the purpose of logging or otherwise recording request metadata. |
| [Content-Type](https://datatracker.ietf.org/doc/html/rfc7231#section-3.1.1.5){: external} | Request, Response | For all responses with bodies, this header MUST be included and indicate the content type of the body. Requests including this header with a content type not supported by the server MUST be rejected with a `415 Unsupported Media Type` status code and appropriate error response model. |
{: caption="Representation headers" caption-side="bottom"}

### Content encoding
{: #content-encoding}

By default, responses SHOULD be encoded with `gzip`. If a request specifies a preference for an
unsupported encoding in the `Accept-Encoding` header, responses MUST NOT be encoded. If the
`Accept-Encoding` header requires that an unsupported encoding be used, requests MUST be rejected
with a `406 Not Acceptable` status code and and appropriate error response model in an unencoded
body.

Encoded requests MAY be supported in cases where clients need to send large request bodies. If
request encoding is supported at all, `gzip` SHOULD be permitted.

### Content location
{: #content-location}

The `Content-Location` response header can provide valuable information to the client, particularly
for state-changing methods. The advantage to providing a `Content-Location` header is that a client
can _always_ depend on this header to provide the permanent location of the content returned in a
response body.

*  For responses to `POST` requests which return a `201` status along with the created resource,
   `Content-Location` SHOULD be included and match `Location`.
*  For responses to `PUT` or `PATCH` requests which modify the resource at the request URI and
   return a `200` status along with the modified resource, `Content-Location` SHOULD be included and
   match the request URI.
*  For responses to `POST`, `PUT`, `PATCH`, or `DELETE` requests which return a `202` status and
   include a resource representing the status of the request, `Content-Location` SHOULD be included
   and contain a URI where this request status may be obtained.
*  For responses to any other `POST`, `PUT`, `PATCH`, or `DELETE` requests which include a resource
   in the response body which may be found at a permanent URI, `Content-Location` SHOULD be included
   and contain this URI.

The body of a response which includes a `Content-Location` header MUST be identical to the body of
the response to a successful `GET` request made to the URI provided in that `Content-Location`
header.

## Negotiation headers
{: #negotiation-headers}

| Header | Type | Description |
| ------ | ---- | ----------- |
| [Accept](https://datatracker.ietf.org/doc/html/rfc7231#section-5.3.2){: external} | Request  | For requests including this header, the server SHOULD attempt to return resources in a format specified as acceptable. Requests including this header with only formats not supported by the server MUST be rejected with a `406` status code and appropriate error response model. This header MUST NOT be required. |
| [Accept-Charset](https://datatracker.ietf.org/doc/html/rfc7231#section-5.3.3){: external} | Request  | For requests including this header, the server SHOULD attempt to return resources in a character set specified as acceptable. If this header is not included, the response MUST use `utf-8`. Requests including this header with only character sets not supported by the server MUST be rejected with a `406` status code and appropriate error response model. This header MUST NOT be required. |
| [Accept-Encoding](https://datatracker.ietf.org/doc/html/rfc7231#section-5.3.4){: external} | Request  | For requests including this header, the server SHOULD attempt to return resources encoded in an acceptable way. If this header is not included, the response SHOULD use `gzip`. See [above](#content-encoding) for more details. This header MUST NOT be required. |
| [Accept-Language](https://datatracker.ietf.org/doc/html/rfc7231#section-5.3.5){: external} | Request  | For requests including this header, the server MAY attempt to return resource in an acceptable language. This header MUST NOT be required. Furthermore, the value of this header MUST NOT prevent a request from succeeding. |
| [Accept-Ranges](https://datatracker.ietf.org/doc/html/rfc7233#section-2.3){: external} | Response | If a resource supports range requests, this header SHOULD be used to indicate what ranges are supported. |
| [Allow](https://datatracker.ietf.org/doc/html/rfc7231#section-7.4.1){: external} | Response | This header MUST be used to indicate allowable request methods in a `405` response. If deemed useful, it MAY be included in any other response. |
| [Upgrade](https://datatracker.ietf.org/doc/html/rfc7230#section-6.7){: external} | Request, Response | This header MAY be supported for requests and included in responses, in keeping with HTTP standards, to negotiate the use of a new protocol on the same connection. This header MUST NOT be supported except for operations which exist specifically to switch protocols[^only-for-protocol-switching]. The successful outcome of an operation that supports this header (either for requests or responses) MUST be a `101 Switching Protocols` status. |
{: caption="Negotiation headers" caption-side="bottom"}

[^only-for-protocol-switching]: That is, services MUST NOT expect or encourage clients to handle
   protocol switching as a part of operations which may result in normal, successful (`2xx`) HTTP
   responses.

## Authentication headers
{: #authentication-headers}

| Header | Type | Description |
| ------ | ---- | ----------- |
| [Authorization](https://datatracker.ietf.org/doc/html/rfc7235#section-4.2){: external} | Request  | The server MUST support this header for any form of authentication. |
| [WWW-Authenticate](https://datatracker.ietf.org/doc/html/rfc7235#section-4.1){: external} | Response | This header MUST be supplied for responses to requests which need to be authenticated to be successful or requests where credentials are supplied but invalid. It MUST be supplied for all responses with a `401` status. |
{: caption="Authentication headers" caption-side="bottom"}

## Validator headers
{: #validator-headers}

| Header | Type | Description |
| ------ | ---- | ----------- |
| [ETag](https://datatracker.ietf.org/doc/html/rfc7232#section-2.3){: external} | Response | The `ETag` header MAY be supplied with the response to any `GET` or `HEAD` request, or with the response to any request that uses a resource's canonical schema. If supported, the `ETag` SHOULD be a quoted lowercase base-36 string, at least 16 characters in length (e.g., `"md9weho39cn2302n"`) and MUST be based on a checksum or hash of the resource that guarantees it will change if the resource changes. The value MUST have a `W/` prefix if strong ETag semantics are not supported. Even if `W/`-prefixed, an `ETag` MUST be guaranteed to change if any properties are changed that are directly mutable by a client. |
| [Last-Modified](https://datatracker.ietf.org/doc/html/rfc7232#section-2.2){: external} | Response | The `Last-Modified` header MAY be supplied with the response to any `GET` or `HEAD` request. The `Last-Modified` MUST be supplied for any resources that support the `If-Modified-Since` or `If-Unmodified-Since` headers for any methods. The `Last-Modified` header MUST contain a valid [HTTP-date](https://datatracker.ietf.org/doc/html/rfc5322#section-3.3){: external} value in GMT (e.g., `Tue, 15 Nov 1994 12:45:26 GMT`) and MUST NOT be a date/time that occurs in the future. |
{: caption="Validator headers" caption-side="bottom"}

### `ETag` support
{: #etag-support}

The `ETag` header MUST be returned for `GET` and `HEAD` operations on a resource that supports
`If-Match` or `If-None-Match` headers for any operation. If the `ETag` header is returned for any
resource within a service, it SHOULD be returned for all resources in the service. If the `ETag`
header is returned for `GET` operations on a resource, it SHOULD be returned on all operations that
return the resource's canonical schema. The `ETag` header MUST NOT be returned for collection
operations, but a resource MAY also include an `etag` property (with the same format and value) in
its canonical schema, facilitating retrieval of `ETag` values when using collection operations. The
`etag` property MUST NOT be included in non-canonical schemas.

### `Last-Modified` support
{: #last-modified-support}

The `Last-Modified` header MUST be returned for `GET` and `HEAD` operations on a resource that
supports `If-Unmodified-Since` or `If-Modified-Since` headers for any operation. If the
`Last-Modified` header is returned for any resource within a service, it SHOULD be returned for all
resources in the service.

## Conditional headers
{: #conditional-headers}

Conditional headers provide a means for consumers to efficiently ensure a cached resource is
up-to-date and use optimistic locking in order to mitigate race conditions.

| Header              | Type    | Description                              |
| ------------------- | ------- | ---------------------------------------- |
| [If-Match](https://datatracker.ietf.org/doc/html/rfc7232#section-3.1){: external} | Request | This header MAY be supported for any request; it is particularly applicable to `POST`, `PUT`, `PATCH`, and `DELETE` requests. If this condition fails, the server MUST send a `412` status and an appropriate error response model. |
| [If-None-Match](https://datatracker.ietf.org/doc/html/rfc7232#section-3.2){: external} | Request | This header MAY be supported for any request; it is particularly applicable to `GET` and `HEAD` requests. If this condition fails for a `GET` or `HEAD` request, the server MUST send a response with a `304` status and no body. If it fails for a request of any other method, the server MUST send a `412` status and an appropriate error response model. |
| [If-Unmodified-Since](https://datatracker.ietf.org/doc/html/rfc7232#section-3.4){: external} | Request | This header SHOULD NOT be supported for any request. It addresses similar needs to the `If-Match` header, but because of its increased precision, documentation SHOULD encourage clients to use `If-Match`. If supported, and this condition fails, the server MUST send a `412` status and an appropriate error response model. If a client supplies any value other than an [HTTP-date](https://datatracker.ietf.org/doc/html/rfc5322#section-3.3){: external} in this header, the server MUST send a `400` status and an appropriate error response model. |
| [If-Modified-Since](https://datatracker.ietf.org/doc/html/rfc7232#section-3.3){: external} | Request | This header SHOULD NOT be supported for any request. It addresses similar needs to the `If-None-Match` header, but because of its increased precision, documentation SHOULD encourage clients to use `If-None-Match`. If supported, and this condition fails for a `GET` or `HEAD` request, the server MUST send a response with a `304` status and no body. If it fails for a request of any other method, the server MUST send a `412` status and an appropriate error response model. If a client supplies any value other than an [HTTP-date](https://datatracker.ietf.org/doc/html/rfc5322#section-3.3){: external} in this header, the server MUST send a `400` status and an appropriate error response model. |
| [If-Range](https://datatracker.ietf.org/doc/html/rfc7233#section-3.2){: external} | Request | This header MAY be supported for requests which support the `Range` header. If this condition fails, the server MUST ignore the `Range` header and send the entire resource. |
{: caption="Conditional headers" caption-side="bottom"}

### Recommended `If-Match` support
{: #recommended-if-match-support}

The `If-Match` header SHOULD be supported:
- Where [required for robustness](/docs/api-handbook?topic=api-handbook-robustness#optimistic-locking).
- For all `POST`[^post-on-resource], `PUT`, `PATCH`, and `DELETE` operations on a resource for which
  an `ETag` header is returned.

[^post-on-resource]: `If-Match` is only applicable to `POST` requests which mutate or operate on an
   existing resource, such as those used for [exceptional
   operations](/docs/api-handbook?topic=api-handbook-methods#post-other-uses).

### Required checks for unsupported conditional headers
{: #required-checks-for-unsupported-conditional-headers}

An unsupported conditional header in a `POST`, `PUT`, `PATCH`, or `DELETE` request SHOULD cause the
request to fail with a `400` status code and an appropriate error response model.

If an `ETag` header is returned for a resource, an unsupported `If-Match` or `If-None-Match` header
in a `POST`, `PUT`, `PATCH`, or `DELETE` request that operates directly on that resource MUST
cause the request to fail with `400 Bad Request` status.

If a `Last-Modified` header is returned for a resource, an unsupported `If-Unmodified-Since` or
`If-Modified-Since` header in a `POST`, `PUT`, `PATCH`, or `DELETE` request that operates directly
on that resource MUST cause the request to fail with `400 Bad Request` status.

An unsupported conditional header in a `GET` or `HEAD` request MAY be ignored.

## CORS headers
{: #cors-headers}

Support for Cross-Origin Resource Sharing (CORS) MAY be implemented by any service. The following guidelines apply to
services which opt to implement support for CORS.

| Header             | Type     | Description                              |
| ------------------ | -------- | ---------------------------------------- |
| [Access-Control-Allow-Origin](https://fetch.spec.whatwg.org/#http-access-control-allow-origin){: external} | Response | This header MUST be returned when a request is made from a white-listed origin. In this case, its value SHOULD match the value of the request's `Origin` header. |
| [Access-Control-Allow-Credentials](https://fetch.spec.whatwg.org/#http-access-control-allow-credentials){: external} | Response | This header SHOULD be included with a value of `true` if cookies and credentials may be used. Otherwise it SHOULD be omitted. |
| [Access-Control-Expose-Headers](https://fetch.spec.whatwg.org/#http-access-control-expose-headers){: external} | Response | This header SHOULD be used to list which response headers should be made accessible to the client apart from `Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `Last-Modified`, and `Pragma`, which are all exposed by default. |
| [Access-Control-Max-Age](https://fetch.spec.whatwg.org/#http-access-control-max-age){: external} | Response | This header SHOULD be included with responses to preflight requests to indicate how long a client may cache the results. |
| [Access-Control-Allow-Methods](https://fetch.spec.whatwg.org/#access-control-allow-methods-response-header){: external} | Response | This header SHOULD be included with responses to preflight requests to indicate what request methods are permissible. |
| [Access-Control-Allow-Headers](https://fetch.spec.whatwg.org/#http-access-control-allow-headers){: external} | Response | This header MAY be included with any responses to preflight requests to indicate what request headers are permissible. It MUST be included in responses to requests including the `Access-Control-Request-Headers` header. |
| [Access-Control-Request-Method](https://fetch.spec.whatwg.org/#http-access-control-request-method){: external} | Request | This header SHOULD be understood and utilized in order to formulated responses to preflight requests. |
| [Access-Control-Request-Headers](https://fetch.spec.whatwg.org/#http-access-control-request-headers){: external} | Request | This header SHOULD be understood and utilized in order to formulate responses to preflight requests. |
| [Origin](https://fetch.spec.whatwg.org/#origin-request-header){: external} | Request | This header MUST be understood and utilized by services implementing support for CORS. |
{: caption="CORS headers" caption-side="bottom"}

## Preference headers
{: #preference-headers}

| Header             | Type     | Description                              |
| ------------------ | -------- | ---------------------------------------- |
| [Prefer](https://datatracker.ietf.org/doc/html/rfc7240#section-2){: external} | Request  | This header SHOULD be supported for `POST`, `PUT`, and `PATCH` requests where it may be expensive (in terms of bandwidth or computation) to return the created or modified resource. If the client supplies `return=minimal`, a successful response MUST have a `201 Created` or `204 No Content` status and include no body. If `return=representation` is supplied, a successful response MUST include the acted-upon resource. Whether the resource is included in the response by default is up to the implementors of the service. |
| [Preference-Applied](https://datatracker.ietf.org/doc/html/rfc7240#section-3){: external} | Response | This header MAY be returned with responses to requests which include a `Prefer` header but is usually not necessary as it is only needed when the client does not have any other clear indicators as to whether its preference was applied. |
{: caption="Preference headers" caption-side="bottom"}

## Context headers
{: #context-headers}

| Header     | Type     | Description |
| ---------- | -------- | ----------- |
| [From](https://datatracker.ietf.org/doc/html/rfc7231#section-5.5.1){: external} | Request  | This header allows a client to provide an email address where the maintainer of the client system may be contacted. Its value MUST NOT affect the outcome of any request. |
| [Referer](https://datatracker.ietf.org/doc/html/rfc7231#section-5.5.2){: external} | Request  | This header allow a client to reference where the request URI was obtained. Its value MUST NOT affect the outcome of any request. |
| [Server](https://datatracker.ietf.org/doc/html/rfc7231#section-7.4.2){: external} | Response | This header SHOULD be provided in all responses and SHOULD contain the public-facing name and version number (including any minor and patch version) of the service responsible for handling the request. |
| [User-Agent](https://datatracker.ietf.org/doc/html/rfc7231#section-5.5.3){: external} | Request  | This header allows a client to report information about itself. It SHOULD be logged with a request, but MAY be otherwise ignored. |
{: caption="Context headers" caption-side="bottom"}

## Control headers
{: #control-headers}

| Header        | Type              | Description                              |
| ------------- | ----------------- | ---------------------------------------- |
| [Age](https://datatracker.ietf.org/doc/html/rfc7234#section-5.1){: external} | Response | This header is used by caching servers to indicate the age of a request. |
| [Cache-Control](https://datatracker.ietf.org/doc/html/rfc7234#section-5.2){: external} | Request, Response | This header specifies directives for caches. It MAY be supported if the presence of caches is anticipated. |
| [Date](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.1.2){: external} | Request, Response | This header represents the time a request or response was originally sent, and MUST be included in responses. When included by requests, the server MUST NOT presume that the client's clock is accurate. |
| [Expect](https://datatracker.ietf.org/doc/html/rfc7231#section-5.1.1){: external} | Request | This header is used in conjunction with the `100` status code and MAY be supported. |
| [Expires](https://datatracker.ietf.org/doc/html/rfc7234#section-5.3){: external} | Response | This header indicates when a response will become stale and should be purged from caches. It MAY be supported if the presence of caches is anticipated. |
| [Host](https://datatracker.ietf.org/doc/html/rfc7230#section-5.4){: external} | Request | This header indicates the host and port for the target URI of the request. If it is omitted or duplicated, the request MUST be rejected with a `400` status and appropriate error response model. |
| [Location](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.2){: external} | Response | This header is required for certain status codes. The semantics for its usage is dictated by the status code it is paired with. |
| [Max-Forwards](https://datatracker.ietf.org/doc/html/rfc7231#section-5.1.2){: external} | Request | This header is used by proxies in conjunction with `TRACE` and `OPTIONS` requests and MAY be ignored by the server. |
| [Pragma](https://datatracker.ietf.org/doc/html/rfc7234#section-5.4){: external} | Request | This header specifies directives for caches. It MAY be ignored. |
| [Range](https://datatracker.ietf.org/doc/html/rfc7233#section-3.1){: external} | Request | This header MAY be supported for large resources where clients might want to obtain only a resource in separate parts. It MUST be ignored for any requests with a method other than `GET`. |
| [Retry-After](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.3){: external} | Response | This header MAY be sent with either a `503` status to indicate when a service is expected to be available again or with any `3xx` status to indicate how long a client should wait before executing a redirect. It MUST contain either a valid [HTTP-date](https://datatracker.ietf.org/doc/html/rfc5322#section-3.3){: external} value or an integer representing a number of seconds. |
| [TE](https://datatracker.ietf.org/doc/html/rfc7230#section-4.3){: external} | Request | This header is used to specify what transfer codings (besides chunked) a client is willing to accept and MAY be ignored. |
| [Vary](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.4){: external} | Response | This header specifies what request fields may affect the outcome of a request. It MAY be supported if the presence of caches is anticipated or to proactively signal support for different content format, encodings, languages, etc.  |
| [Warning](https://datatracker.ietf.org/doc/html/rfc7234#section-5.5){: external} | Response | This header gives additional information that might not be conveyed in the status code and doesn't represent a total failure. It MAY be supported, particularly if the presence of caches is anticipated. |
{: caption="Control headers" caption-side="bottom"}

## Tracing headers
{: #tracing-headers}

Tracing headers allow services to log and propagate the context of individual requests and requests
that are part of a larger context.

A value for one of these headers MAY include ASCII alphanumerics and any of following segment
separators: space (` `), comma (`,`), hyphen, (`-`), and underscore (`_`) and MAY have a length up
to 1024 bytes. A value MUST be considered invalid and MUST be ignored if that value includes any
other character or is longer than 1024 bytes.

A value for one of these headers SHOULD be considered invalid and SHOULD be ignored if that value
is known to be of static origin (such as an all-zero UUID) or insufficiently unique (such as a
value with fewer than 8 characters).

A value for the `X-Correlation-ID` header MUST be considered invalid and MUST be ignored if it is
supplied by a client known to be untrusted (that is, a client not written and operated by IBM
itself). This directive does not apply to a value for `X-Request-ID`.

If a value for one of these headers is invalid or not supplied in a request, the service MUST
generate a random (version 4) UUID to use in place of a client-supplied value.

| Header                | Type     | Description                              |
| --------------------- | -------- | ---------------------------------------- |
| X-Request-ID | Request, Response | The supplied or generated value of this header MUST be logged for a request and repeated in a response header for the corresponding response. The same value MUST NOT be used for downstream requests or between retries of those requests. Documentation for a service SHOULD encourage all client developers to supply this header with a random UUID. |
| X-Correlation-ID | Request, Response | The supplied or generated value of this header MUST be logged for a request and repeated in a response header for the corresponding response. The same value MUST be used for downstream requests and retries of those requests. Documentation for a service SHOULD encourage internal client developers to supply this header with a random UUID or (if available) a valid, trusted upstream value. |
{: caption="Tracing headers" caption-side="bottom"}

## Rate limiting headers
{: #rate-limiting-headers}

The standard `Retry-After` header MUST be returned in a response with status code 429
for any implementation of rate limiting[^rate-limiting-retry-after].

| Header                | Type     | Description                              |
| --------------------- | -------- | ---------------------------------------- |
| Retry-After | Response | If rate limiting is active, this header MUST indicate how long the user agent should wait before making a follow-up request. The value of this field can be either an HTTP-date or a number of seconds to delay after the response is received. |
{: caption="Retry-After header" caption-side="bottom"}

The following headers, while not a part of the HTTP specification, are common practice[^rate-limiting-common-practice]
and MAY be used for any implementation of rate limiting.

| Header                | Type     | Description                              |
| --------------------- | -------- | ---------------------------------------- |
| X-RateLimit-Limit | Response | If rate limiting is active, this header MUST indicate the number of requests permitted per hour. |
| X-RateLimit-Remaining | Response | If rate limiting is active, this header MUST indicate the number of requests remaining in the current rate limit window. |
| X-RateLimit-Reset | Response | If rate limiting is active, this header MUST indicate the time at which the current rate limit window resets, as a UNIX timestamp. |
{: caption="Rate limiting headers" caption-side="bottom"}

## IBM Cloud headers

| Header | Type | Description |
| ------ | ---- | ----------- |
| [IBM-API-Dry-Run](#dry-run) | Request | This header MAY be supported for requests that allow simulation of the operation. This header MUST NOT be required. See [below](#dry-run) for more details. |
| [IBM-API-Dry-Run](#dry-run) | Response | If the operation supports dry-run and the caller requested a supported dry-run type, this header MUST be included with the caller's requested value. Otherwise, this header MUST NOT be included. |
| [IBM-API-Version](/docs/api-handbook?topic=api-handbook-changes-overview#date-based-api-versioning) | Request | This header MUST be supplied to services that support it. The value of this header MUST be the version of the API requested by the client. |

### Dry-run
{: #dry-run}

The `IBM-API-Dry-Run` header allows an operation to be simulated to a varying extent. This is
useful in a number of situations, including:
- An interactive client which needs to present different choices to the user depending on their
  authorization level, consumption against quota, and other run-time criteria.
- An interactive client which does not wish to duplicate complex or computationally expensive
  server-side validation logic.
- A client which wishes to short-circuit computationally expensive work by performing a dry-run with
  mock input. Only if the dry-run succeeds will the client eat the cost of creating the real input.
- A client written to explore an API without creating, updating, or deleting any persisted
  resources.
- An automated API contract test which doesn't require any cleanup steps.

If an operation supports `IBM-API-Dry-Run`, it MUST support the `validate` value and MAY also
support the `mock` value. When a request is made with a supported value, the response MUST reflect
that value back in an identical `IBM-API-Dry-Run` response header. If a request includes
`IBM-API-Dry-Run` for an operation that does not support the header, or does not recognize or
support the value provided, the service MUST respond with `400 Bad Request`.

Any request including the `IBM-API-Dry-Run` header MUST NOT cause any side effects observable in
other (concurrent or subsequent) requests to the API. These requests MUST still generate logs and
auditing events as usual, but any log or event that is directly tied to the request MUST preserve
the `IBM-API-Dry-Run` header and value alongside the rest of the information about the request.

Even if a dry-run request succeeds, due to limitations of the dry-run validation or changes in the
surrounding environment, a subsequent otherwise-identical actual may fail. As such, APIs supporting
this header SHOULD advise clients on how to robustly use dry-run.{:note}

### Validation

When a request includes the `IBM-API-Dry-Run` header with the `validate` value, the service MUST
perform best-effort validation on the request. If a problem is found with the request, the service
MUST respond with an error response identical to the one that would be supplied for a normal
request. If no problems are found, the service MUST respond with `204 No Content`.

Validation MUST include authentication and coherency checks for path parameters, query parameters,
other request headers, and the request body. That is, the service MUST verify the authentication
token provided with the request, and MUST validate that values provided in the request are
well-formed, conform the defined schemas, and are self-consistent.

Validation SHOULD include authorization, quota checks, and conflict checks. That is, the service
SHOULD verify that:
- The authenticated caller is authorized to perform the requested operation.
- The requested operation would not cause account quotas to be exceeded.
- The requested operation would not conflict with existing system state.

Validation MAY include capacity checks. That is, a service MAY verify whether physical capacity
exists to fulfill the request.

The API documentation MUST specify which forms of validation are performed. If the validation
differs depending on the operation, it MUST be documented on a per-operation basis.

### Mocking

When a request includes the `IBM-API-Dry-Run` header with the `mock` value, and the operation
supports it, the same validation used for the `validate` value MUST be performed, and the same error
responses MUST be provided as applicable. However, if the validation is successful, the service MUST
respond with the operation's normal success code (such as `201 Created` or `202 Accepted`), normal
headers, and the normal response schema (if any), with realistic data mocked as needed.

When returning mock data, the server MUST conform to the operation's response schema (in particular,
all required properties MUST be present). The mock data SHOULD NOT conflict with an actual resource
(such as having a `crn` or `name` that actually exists), and SHOULD adhere to industry best
practices for example values, such as domain names that end in `example.com` and IP addresses from
`192.0.2.0/24`.

## Custom headers
{: #custom-headers}

There are times where service-specific functionality may need to be implemented with custom headers.
In these cases, support for custom request or response headers MAY be implemented. However, custom
headers SHOULD NOT be required for basic usage of a given API.

When naming custom headers, care should be taken to disambiguate them as much as possible. They
SHOULD NOT be prefixed with `X-`. If a custom header is by nature limited in scope to IBM-specific
products and services it SHOULD begin with `IBM-` or `IBM-PRODUCT-` where `PRODUCT` is, for example,
`Watson`. Custom header names SHOULD be constructed from title-cased, hyphen-separated words.
However, as with all headers, they MUST be handled in a case-insensitive way.

## Request headers as query parameters
{: #request-headers-as-query-parameters}

In cases where a request header may be inconvenient for a client to supply request header
functionality MAY be duplicated as query parameters. In these cases, the name of the query parameter
should be as close to the name as the header as possible while still complying with all the rules
for query parameter names. For example, the request header `If-Match` would be exposed as the query
parameter `if_match`.

[^rate-limiting-retry-after]: The `Retry-After` header is defined in [IETF RFC
   7231](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.3){: external}. The rate-limiting
   implementation in CloudFlare does provide the `Retry-After` header in `429` responses.

[^rate-limiting-common-practice]: These headers are used by
   [Twitter](https://developer.x.com/en/docs/rate-limits){: external},
   [GitHub](https://docs.github.com/en/rest/rate-limit)){: external},
   [Atlassian](https://developer.atlassian.com/server/hipchat/hipchat-rest-api-rate-limits/){: external},
   and others.
