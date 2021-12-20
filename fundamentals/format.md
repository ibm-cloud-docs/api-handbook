---

copyright:
  years: 2019
lastupdated: "2019-12-01"

subcollection: api-handbook

---

{:external: .external}
{:important: .important}

# Format
{: #format}

APIs MUST support JSON by default for sending and receiving data in request and response
bodies. Other formats SHOULD NOT be supported except to meet a compelling
industry-specific need.

## Object encapsulation
{: #object-encapsulation}

All JSON data MUST be structured in an object at the top level; arrays MUST NOT be returned as the
top-level structure in a response body.[^never-use-root-arrays]

## Request body constraints
{: #request-body-constraints}

Services MUST have a documented and enforced maximum payload size for request bodies. Requests with
bodies larger than the limit MUST be rejected with a `413 Payload Too Large` status code and
appropriate error response model.

If the request body size can be determined from the request headers, the request MUST be rejected
prior to any processing of the payload[^processing]. For chunked transfer or other cases where the
request body size cannot be predetermined, the request SHOULD be rejected as soon as it has been
determined to exceed the limit. 

[^processing]: Specifically, a service SHOULD NOT attempt to parse the JSON in a payload prior to
   checking and enforcing the size limit.

## Content-Type behavior
{: #content-type-behavior}

If a request `Accept` header is either not provided or matches[^matching-accept-content-type]
`application/json`, the response MUST be JSON and its `Content-Type` header MUST be
`application/json`.

## JSON processing
{: #json-processing}

### Case insensitivity
{: #case-insensitivity}

Field names in JSON objects in a request payload SHOULD NOT be case-normalized to support case
insensitivity; a field that does not match the case of a defined field but otherwise matches its
name SHOULD be treated as any other extraneous input[^field-case-normalization].

However, field name case normalization MAY be supported for backward compatibility with existing
clients.

[^field-case-normalization]: Case normalization is often an error-prone process, however simple
   it may seem. One problem is that different standard libraries may not agree on the
   lowercase-equivalent value for a particular string. For example, `"İstanbul".ToLowerCase()` in
   JavaScript yields `i̇stanbul` (note the two dots over the first character), but
   `strings.ToLower("İstanbul")` in Go is more aware of locale-specific rules and yields `istanbul`.
   If the code that validates and the code that actually uses a particular field disagree on the
   normalization of its name, it could lead to a bug that could be exploited to validate one value
   and use another.
  
### Ambiguous JSON data
{: #ambiguous-json-data}

Duplicate field names MUST NOT exist in any one JSON object[^one-json-object] contained in a JSON
response. If duplicate field names exist in an object within a request payload, the request MUST be
rejected[^duplicate-json-field-names] with a `400` status and appropriate error response model.

Be aware that the JSON specification itself does not strictly prohibit duplicate field names in a
single object and many unmarshalling libraries may silently choose from among duplicated fields.
{: important}

If a service supports field name [case insensitivity](#case-insensitivity), field names MUST be
normalized prior to validating uniqueness.

[^one-json-object]: Using the same field name in different objects within a single JSON payload is
   perfectly acceptable.

[^duplicate-json-field-names]: Silent support for ambiguously duplicated fields in JSON documents
   increases the risk that a malicious client could craft a JSON document that bypasses critical
   authorization or validation checks.

## Cross-origin support
{: #cross-origin-support}

If a service has a need to support cross-origin requests, CORS headers SHOULD be supported to enable
this. The JSONP format SHOULD NOT be supported.

[^never-use-root-arrays]: This design allows for top-level metadata to be added later and prevents a
   [cross-site scripting
   vulnerability](http://haacked.com/archive/2008/11/20/anatomy-of-a-subtle-json-vulnerability.aspx/){: external}.

[^matching-accept-content-type]: It's very important to note that API implementations should _not_
   parse `Accept` headers with a simple string search for `application/json`, as wildcards are often
   used. It's also important for matching to be
   [case-insensitve](https://datatracker.ietf.org/doc/html/rfc2045#section-5.1){: external}, as MIME
   types are lowercase by convention only.
