---

copyright:
  years: 2020, 2021
lastupdated: "2021-06-30"

subcollection: api-handbook

---

# Changes overview
{: #changes-overview}

Services MUST have semantic **service versioning** and MAY support date-based **API versioning**.

## Semantic service versioning
{: #semantic-service-versioning}

Services MAY have a major, minor, and patch version, following the industry conventions on
[semantic versioning](https://semver.org/):

> Given a version number MAJOR.MINOR.PATCH, increment the:
>
> MAJOR version when you make incompatible API changes,
> MINOR version when you add functionality in a backwards-compatible manner, and
> PATCH version when you make backwards-compatible bug fixes.

If surfaced, this version SHOULD be exposed in the `Server` response header.

### Major version
{: #major-version}

The major version of a service MUST be represented in its URL path,
[as detailed](/docs/api-handbook/design/uris.html#version) in the URI section. A service MAY support
multiple major versions concurrently.

Incrementing of the major version MUST NOT occur frequently, and SHOULD occur only as a part of a
significant reconception of a service. A new major version of a service MAY support an entirely
separate resource scope[^resource-scope] that does not support resources created or managed with
prior versions.

### Minor and patch versions
{: #minor-and-patch-versions}

The minor and patch version MUST NOT be client-selectable at runtime for a service deployment.

## Date-based API versioning
{: #date-based-api-versioning}

If client-selectable versions of API behavior are needed, services MAY support the `version` query
parameter, accepting a date-based API version in the format `YYYY-MM-DD`. The `version` query
parameter MUST NOT have any alternative use.

### Supported version dates
{: #supported-version-dates}

Each major version of a service MUST support a single range of contiguous API version dates.

A non-deprecated major version of a service MUST support up to (but not beyond) the current date
(from `00:00 UTC` of that date).

From an API client developer's perspective, any date in the supported range is equally valid. From
a service developer's point of view, version dates provided by the client SHOULD be canonicalized
to a small number of "inflection" dates — version dates at which the behavior of the service
actually changes. This could be considered an exception to the general policy to avoid [unnecessary
input canonicalization](/docs/api-handbook?topic=api-handbook-robustness#input-canonicalization).

When a client provides a version date between two inflection dates (or after the most recent
inflection date), the behavior of the API MUST match the behavior defined for the closest _prior_
inflection date. For example, if the behavior of an API mostly recently changed for the
`2021-06-01` version date and a client provides `2021-06-30`, that service MUST behave as if
`2021-06-01` was provided.

An update to the service MAY move the oldest supported API version date forward in accordance with
deprecation policies. A deprecated major version SHOULD establish a fixed last supported date when
its deprecation is announced.

### Version dates
{: #version-dates}

An update to a service MAY support new API behavior specific to a new API version date, or
across version dates, depending on the
[compatibility](/docs/api-handbook/changes/compatibility.html) of the change. The service
implementation supporting a change specific to a new _API version date_ (e.g., `2019-11-19`) MUST be
released prior to that date.

### Client developer directives
{: #client-developer-directives}

Client developers SHOULD be encouraged to use the current date as a version date during development
and MUST be encouraged to use a fixed value for testing and production release.

[^resource-scope]: Where a "resource scope" is the enclosing environment for resources, where
  resources coexist and relate to each other.
