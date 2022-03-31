---

copyright:
  years: 2022
lastupdated: "2022-03-22"

subcollection: api-handbook

---

{:external: .external}

# Standard behavior
{: #standard-behavior}

Certain HTTP behaviors are expected to be implemented uniformly and clearly understood by client
applications. This section offers guidance on specific behaviors that MAY be considered standard and
MAY be omitted from an API definition.

## Headers
{: #standard-headers}

Support for following headers MAY be omitted from the documentation for each operation. Unique
aspects of a service's support for these headers SHOULD be documented in the supporting materials
for the API reference. The required behavior of these headers is defined in the [Headers][headers]
section.

If a header has unique importance or behavior for an operation, that header SHOULD be documented
for the operation along with a description of its salient behavior. If a header with normally
uniform support necessarily has uneven support in a service (for example, the `Authorization`
header in an API that provides authentication), the header SHOULD be documented for the specific
operations that do support it.

[headers]: /docs/api-handbook?topic=api-handbook-headers

| Header              | Explanation                                                                |
| ------------------- | -------------------------------------------------------------------------- |
| `Accept-Charset`    | Uniform support, described as needed in supporting materials               |
| `Accept-Encoding`   | Uniform support, described as needed in supporting materials               |
| `Accept`            | Defined by the [request body object][request-body-object]{: external}      |
| `Age`               | Standard HTTP behavior                                                     |
| `Allow`             | Standard HTTP behavior                                                     |
| `Authorization`     | Standard HTTP behavior for [authentication][authentication]                |
| `Cache-Control`     | Standard HTTP behavior                                                     |
| `Content-Encoding`  | Standard HTTP behavior                                                     |
| `Content-Language`  | Standard HTTP behavior                                                     |
| `Content-Location`  | Standard HTTP behavior                                                     |
| `Content-Type`      | Defined by the [response object][response-object]{: external}              |
| `Date`              | Standard HTTP behavior                                                     |
| `Host`              | Standard HTTP behavior                                                     |
| `Location`          | Standard HTTP behavior                                                     |
| `Referer`           | Standard HTTP behavior                                                     |
| `Retry-After`       | Standard HTTP behavior                                                     |
| `Server`            | Standard HTTP behavior                                                     |
| `User-Agent`        | Standard HTTP behavior                                                     |
| `X-Correlation-ID`  | Uniform support for [tracing headers][tracing-headers]                     |
| `X-Request-ID`      | Uniform support for [tracing headers][tracing-headers]                     |
{: caption="Implicitly supported headers" caption-side="bottom"}

[request-body-object]: https://spec.openapis.org/oas/v3.0.3#request-body-object
[request-body-object]: https://spec.openapis.org/oas/v3.0.3#response-object
[tracing-headers]: /docs/api-handbook?topic=api-handbook-headers#tracing-headers

## Status codes
{: #standard-status-codes}

Support for following status codes MAY be omitted from the documentation for each operation. Unique
aspects of a service's support for these status codes SHOULD be documented in the supporting
materials for the API reference. The required behavior of these status codes is defined in the
[Status codes][status-codes] section.

[status-codes]: /docs/api-handbook?topic=api-handbook-status-codes

| Status code | Explanation                                                                        |
| ----------- | ---------------------------------------------------------------------------------- |
| `304`       | Standard HTTP behavior for [conditional requests][conditional-requests]            |
| `401`       | Standard HTTP behavior for [authentication][authentication]                        |
| `403`       | Standard HTTP behavior for [authorization][authorization]                          |
| `404`       | Standard HTTP behavior                                                             |
| `405`       | Standard HTTP behavior                                                             |
| `406`       | Standard HTTP behavior                                                             |
| `408`       | Standard HTTP behavior                                                             |
| `412`       | Standard HTTP behavior for [conditional requests][conditional-requests]            |
| `413`       | Standard HTTP behavior for [request body constraints][request-body-constraints]    |
| `414`       | Standard HTTP behavior for [URI constraints][uri-constraints]                      |
| `415`       | Standard HTTP behavior                                                             |
| `429`       | Standard HTTP behavior for [rate limiting][rate-limiting]                          |
| `500`       | Standard HTTP behavior                                                             |
| `501`       | Standard HTTP behavior                                                             |
| `503`       | Standard HTTP behavior                                                             |
| `505`       | Standard HTTP behavior                                                             |
{: caption="Implicitly supported status codes" caption-side="bottom"}

[conditional-requests]: /docs/api-handbook?topic=api-handbook-headers#conditional-headers
[authentication]: /docs/api-handbook?topic=api-handbook-authentication
[authorization]: /docs/api-handbook?topic=api-handbook-authorization
[request-body-constraints]: /docs/api-handbook?topic=api-handbook-format#request-body-constraints
[uri-constraints]: /docs/api-handbook?topic=api-handbook-uris#constraints
[rate-limiting]: /docs/api-handbook?topic=api-handbook-headers#rate-limiting-headers

The status codes `500` and `503` SHOULD NOT be documented for any operation. Because these status
codes are intended to represent transient runtime failures, they MUST NOT be considered a part of
normal API behavior.
