---

copyright:
  years: 2020
lastupdated: "2020-02-13"

subcollection: api-handbook

---

# Changes overview

Services MUST have semantic **service versioning** and MAY support date-based **API versioning**.

## Semantic service versioning

Services MAY have a major, minor, and patch version, following the industry conventions on
[semantic versioning](https://semver.org/):

> Given a version number MAJOR.MINOR.PATCH, increment the:
>
> MAJOR version when you make incompatible API changes,
> MINOR version when you add functionality in a backwards-compatible manner, and
> PATCH version when you make backwards-compatible bug fixes.

If surfaced, this version SHOULD be exposed in the `Server` response header.

### Major version

The major version of a service MUST be represented in its URL path,
[as detailed](/docs/api-handbook/design/uris.html#version) in the URI section. A service MAY support
multiple major versions concurrently.

Incrementing of the major version MUST NOT occur frequently, and SHOULD occur only as a part of a
significant reconception of a service. A new major version of a service MAY support an entirely
separate resource scope[^resource-scope] that does not support resources created or managed with
prior versions.

### Minor and patch versions

The minor and patch version MUST NOT be client-selectable at runtime for a service deployment.

## Date-based API versioning

If client-selectable versions of API behavior are needed, services MAY support the `version` query
parameter, accepting a date-based API version in the format `YYYY-MM-DD`. The `version` query
parameter MUST NOT have any alternative use.

### Supported version dates

Each major version of a service MUST support a single range of contiguous API version dates.

A non-deprecated major version of a service MUST support up to (but not beyond) the current date
(from `00:00 UTC` of that date). An update to the service MAY move the oldest supported API version
date forward in accordance with deprecation policies.

A deprecated major version SHOULD establish a fixed last supported date when its deprecation is
announced.

### Version dates

An update to a service MAY support new API behavior specific to a new API version date, or
across version dates, depending on the
[compatibility](/docs/api-handbook/changes/compatibility.html) of the change. The service
implementation supporting a change specific to a new _API version date_ (e.g., `2019-11-19`) MUST be
released prior to that date.

### Client developer directives

Client developers SHOULD be encouraged to use the current date as a version date during development
and MUST be encouraged to use a fixed value for testing and production release.

[^resource-scope]: Where a "resource scope" is the enclosing environment for resources, where
  resources coexist and relate to each other.
