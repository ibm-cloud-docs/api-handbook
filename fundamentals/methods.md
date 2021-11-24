---

copyright:
  years: 2019, 2021
lastupdated: "2021-06-30"

subcollection: api-handbook

---

{:external: .external}

# Methods
{: #methods}

There are seven methods that MAY be supported by a resource URI. A summary table of each method's
basic attributes is listed below, followed by detailed guidelines for the use of each method.

## Summary
{: #summary}

| Method  | Request body | Response body | Safe | Idempotent |
| ------- | ------------ | ------------- | ---- | ---------- |
| GET     | No           | Yes           | Yes  | Yes        |
| HEAD    | No           | No            | Yes  | Yes        |
| POST    | Yes          | Yes           | No   | No         |
| PUT     | Yes          | Yes           | No   | Yes        |
| PATCH   | Yes          | Yes           | No   | No         |
| DELETE  | No           | Yes           | No   | Yes        |
| OPTIONS | No           | Yes           | Yes  | Yes        |
{: caption="HTTP methods" caption-side="bottom"}

## GET
{: #get}

The response to a successful `GET` request SHOULD contain the state of a resource or collection of
resources, based on the request URI. A `GET` request MUST be safe and therefore MUST NOT have side
effects[^safe-side-effects]. If a `GET` request contains a body, the body MUST be ignored and
therefore MUST NOT cause an error.[^get-request-body]

## HEAD
{: #head}

The response to a successful `HEAD` request MUST contain the exact set of headers that the response
to an otherwise-identical `GET` request would contain, but MUST NOT include a response
body.[^head-behavior] This can be used for a client to make optimizations if it needs only the
information contained in response headers.

If `HEAD`-specific optimizations are not feasible, the handling of a `HEAD` request MAY be
implemented internally by using the same code path as the handling of the corresponding `GET`
request and subsequently discarding the response body. As with a `GET` request, a `HEAD` request
MUST be safe and its body MUST be ignored.

## POST
{: #post}

### For creation
{: #post-for-creation}

The `POST` URI used to create a resource SHOULD be the same as the `GET` URI for listing resources
of the same kind.[^uri-accepting-post] For example, if `GET /users` lists all users, and users can
be created by a client, then `POST /users` SHOULD create a user.

The response to a `POST` request which successfully and synchronously creates a resource MUST have
the status code `201 Created` and a `Location` header with the URI of the newly-created
resource[^post-create-result] and SHOULD include the created resource in the response body. If
including the resource is expensive (in terms of bandwidth or computation), [preference
headers](/docs/api-handbook?topic=api-handbook-headers#preference-headers) SHOULD be supported to
give the client control over whether the resource is included.

Fields which are not mutable or not recognized MUST result in a `400` status code and appropriate
error response model.

### Other uses
{: #post-other-uses}

Since `POST` requests are the only avenue for handling unsafe, non-idempotent, or safe and
body-bearing[^safe-with-body] operations that don't fit into another category, a `POST` request MAY
invoke such an operation, even if it does not result in the explicit creation of a resource.

In this case, the request body MAY be permitted to be absent or have a format unrelated to any
resource which can be retrieved using a `GET` request. When such a request is successful, it SHOULD
return a `200 OK` status (for a response including a description of the result) or a `204 No
Content` status.[^post-not-for-create]

[^safe-with-body]: Such as a request supporting a complex query in a JSON payload.

## PUT
{: #put}

A successful `PUT` request SHOULD replace an existing resource or, in specific cases, create a new
resource.

The resource impacted by a `PUT` request MUST be located at the URI used in the request.[^put-uri]
For example, if `PUT /users/bob` updates a user, `GET /users/bob` MUST retrieve the same user.

When a `PUT` request is successfully and synchronously used to replace an existing resource, a
status of `200 OK` MUST be returned.[^put-update-success]

Fields which are not mutable or not recognized MUST result in a `400` status code and appropriate
error response model.

In a scenario where the URI of a new resource can be inferred in advance by the client, a `PUT`
request MAY create a new resource. In this case, if the creation is performed synchronously, the
response MUST return a `201 Created` status.[^put-create-success]

The response to a successful, synchronous `PUT` request SHOULD include the created or replaced
resource in the response body. If doing so is expensive (in terms of bandwidth or computation),
[preference headers](/docs/api-handbook?topic=api-handbook-headers#preference-headers) SHOULD be
supported to give the client control over whether the resource is included.

## PATCH
{: #patch}

A `PATCH` request SHOULD specify new values for one or more fields on a resource and SHOULD be
considered successful if each field was updated to, or already contained, the value specified in the
request. The resource updated by a `PATCH` request MUST be located at the URI used in the
request.[^patch-uri] Unlike with a `PUT` request, services SHOULD NOT require a `PATCH` request to
reference all mutable fields.

Services MAY support either or both of the below formats for `PATCH` requests:

*  [JSON merge patch format (RFC 7396)](https://datatracker.ietf.org/doc/html/rfc7396){: external}
*  [JSON patch format (RFC 6902)](https://datatracker.ietf.org/doc/html/rfc6902){: external}

The JSON merge patch format (RFC 7396) MAY be supported for requests with a content type of
`application/json` or without an explicit content type. However, if this format is supported at all,
it MUST be supported for requests with a content type of `application/merge-patch+json`.

The JSON patch format (RFC 6902) MAY be supported for requests with a content type of
`application/json-patch+json`. This format MUST NOT be supported for requests of any other content
type.

The response to a `PATCH` request which successfully and synchronously updates a resource MUST have
a status of `200 OK` and SHOULD include the updated resource in the response body. If returning the
resource is expensive (in terms of bandwidth or computation), [preference
headers](/docs/api-handbook?topic=api-handbook-headers#preference-headers) SHOULD be supported to
give the client control over whether the resource is included.

A `PATCH` request referencing fields which are not mutable or not recognized MUST be rejected by
the service with a `400` status code and appropriate error response model.

## DELETE
{: #delete}

A successful `DELETE` request SHOULD remove a resource. The resource deleted by a `DELETE` request
MUST be located at the URI used in the request. If a `DELETE` request contains a body, the body MUST
be ignored and therefore MUST NOT cause an error.[^delete-request-body]


[^safe-side-effects]: It's worth noting that the side effects referred to here are ones visible to
   the user and impacting user data. Side effects such as logging and metric collection are
   permissible for any request.

[^get-request-body]: RFC 2616 says that request bodies [should be
   ignored](https://datatracker.ietf.org/doc/html/rfc2616#section-4.3){: external} for methods that
   do not specify the body as part of the request semantics, and the `GET` method's semantics [only
   include the request URI](https://datatracker.ietf.org/doc/html/rfc2616#section-9.3){: external}.

[^head-behavior]: This behavior is [required by RFC
   2616](https://datatracker.ietf.org/doc/html/rfc2616#section-9.4){: external}.

[^uri-accepting-post]: RFC 2616 [requires
   that](https://datatracker.ietf.org/doc/html/rfc2616#section-9.5){: external} the "posted entity
   is subordinate to that URI in the same way that a file is subordinate to a directory containing
   it."

[^post-create-result]: These two requirements [are
   explicit](https://datatracker.ietf.org/doc/html/rfc2616#section-9.5){: external} in RFC 2616 for
   `POST` requests which create new resources.

[^post-not-for-create]: This use case and the success statuses permitted in it [are
   outlined](https://datatracker.ietf.org/doc/html/rfc2616#section-9.5){: external} in RFC 2616.

[^put-uri]: RFC 2616 requires [says
   that](https://datatracker.ietf.org/doc/html/rfc2616#section-9.6){: external} the "PUT method
   requests that the enclosed entity be stored under the supplied Request-URI."

[^put-update-success]: RFC 2616 [requires
   that](https://datatracker.ietf.org/doc/html/rfc2616#section-9.6){: external} a successful
   resource update by a `PUT` request return either a `200 OK` status or a `204 No Content` status.
   Since this handbook requires that a successful `PUT` request of any kind return the resource as
   represented by a `GET` request to the same URI, only a `200 OK` status is appropriate in this
   case.

[^put-create-success]: [As
   required](https://datatracker.ietf.org/doc/html/rfc2616#section-9.6){: external} by RFC 2616.

[^patch-uri]: RFC 5789
   [states](https://datatracker.ietf.org/doc/html/rfc5789#section-2){: external}, "The PATCH method
   requests that a set of changes described in the request entity be applied to the resource
   identified by the Request-URI."

[^delete-request-body]: Like `GET` requests, `DELETE` requests do not include the request body in
   the [semantics of the
   request](https://datatracker.ietf.org/doc/html/rfc2616#section-9.7){: external}.
