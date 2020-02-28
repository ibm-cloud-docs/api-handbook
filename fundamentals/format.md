---

copyright:
  years: 2019
lastupdated: "2019-12-01"

subcollection: api-handbook

---

# Format

APIs MUST support JSON by default for sending and receiving data in request and response
bodies. Other formats SHOULD NOT be supported except to meet a compelling
industry-specific need.

## Object encapsulation

All JSON data MUST be structured in an object at the top level; arrays MUST NOT be returned as the
top-level structure in a response body.[^never-use-root-arrays]

## Content-Type behavior

If a request `Accept` header is either not provided or matches[^matching-accept-content-type]
`application/json`, the response MUST be JSON and its `Content-Type` header MUST be
`application/json`.

## Cross-origin support

If a service has a need to support cross-origin requests, CORS headers SHOULD be supported to enable
this. The JSONP format SHOULD NOT be supported.

[^never-use-root-arrays]: This design allows for top-level metadata to be added later and prevents a
  [cross-site scripting
  vulnerability](http://haacked.com/archive/2008/11/20/anatomy-of-a-subtle-json-vulnerability.aspx/).

[^matching-accept-content-type]: It's very important to note that API implementations should _not_
  parse `Accept` headers with a simple string search for `application/json`, as wildcards are often
  used. It's also important for matching to be
  [case-insensitve](https://tools.ietf.org/html/rfc2045#section-5.1), as MIME types are lowercase by
  convention only.
