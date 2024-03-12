---

copyright:
  years: 2020, 2021
lastupdated: "2021-06-30"

subcollection: api-handbook

---

{:external: .external}

# Changes overview
{: #changes-overview}

Services MUST have semantic **service versioning** and MAY support date-based **API versioning**.

## Semantic service versioning
{: #semantic-service-versioning}

Services MAY have a major, minor, and patch version, following the industry conventions on
[semantic versioning](https://semver.org/){: external}:

> Given a version number MAJOR.MINOR.PATCH, increment the:
>
> MAJOR version when you make incompatible API changes,
> MINOR version when you add functionality in a backwards-compatible manner, and
> PATCH version when you make backwards-compatible bug fixes.

If surfaced, this version SHOULD be exposed in the `Server` response header.

### Major version
{: #major-version}

The major version of a service MUST be represented in its URL path, [as
detailed](/docs/api-handbook?topic=api-handbook-uris#version) in the URI section. 

Incrementing of the major version:
- MUST NOT occur unless the change results in:
  - Existing resources no longer being represented in the updated resource modeling
  - New resources using the change cannot be safely understood by existing clients
- SHOULD NOT occur more than once a year
- SHOULD NOT occur except as a part of a significant reconception of a service.

For example, a change that makes the [`key`
resource](/docs/api-handbook?topic=api-handbook-resources) regional would require a new major
version, since the new regional keys cannot be safely understood by existing clients, which expected
all keys to be zonal, tied to their `zone` property. Instead, regional keys would need to be placed
in a new `/v2/keys` resource scope, with a migration path to migrate zonal keys from the `/v1/keys`
resource scope into the `/v2/keys` resource scope. (Depending on the broader business and
technological constraints, that migration might preserve the zonal nature of the migrated key, or
promote it to a regional resource.)

A new major version of a service MUST use an entirely separate resource scope[^resource-scope] that
does not support resources created or managed with prior versions. Similarly, while a service MAY
support multiple major versions concurrently (and MUST do so during a transition window), access to a
given resource scope is by definition restricted to one major version. Custom operations MAY be
provided to allow users to perform a one-way migration from a previous version, facilitating a customer
to upgrade their existing resources to a new version prior to adopting that new version.

Incrementing a major version should be a tool of last resort. Before introducing a new version,
carefully consider other possibilities such as introducing a new resource type, or providing tooling
and guidance to allow existing clients to safely manage the transition within the existing major
version.

### Minor and patch versions
{: #minor-and-patch-versions}

The minor and patch version MUST NOT be client-selectable at runtime for a service deployment.

## Date-based API versioning
{: #date-based-api-versioning}

If client-selectable versions of API behavior are needed, services SHOULD support the `IBM-API-Version`
request header[^version-parameter-deprecated], accepting a date-based API version in the format
`YYYY-MM-DD`. The `IBM-API-Version` header MUST NOT have any alternative use. Services that
support the `IBM-API-Version` header MUST return `400 Bad Request` if the header is not provided by
the client, or if the provided value is not within the range supported by the service.

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

An update to a service MAY support new API behavior specific to a new API version date, or across
version dates, depending on the
[compatibility](/docs/api-handbook?topic=api-handbook-change-compatibility) of the change. The
service implementation supporting a change specific to a new _API version date_ (e.g.,
`2019-11-19`) MUST be released prior to that date.

### Client developer directives
{: #client-developer-directives}

Client developers SHOULD be encouraged to use the current date as a version date during development
and MUST be encouraged to use a fixed value for testing and production release.

[^resource-scope]: Where a "resource scope" is the enclosing environment for resources, where
   resources coexist and relate to each other.

[^version-parameter-deprecated]: A previous version of the API Handbook recommended that a `version`
   query parameter be used instead. This is no longer recommended for APIs except to maintain
   backward-compatibility.
