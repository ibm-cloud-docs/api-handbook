---

copyright:
  years: 2021
lastupdated: "2019-12-01"

subcollection: api-handbook

---
# Robustness

## Tradeoffs of the robustness principle

Jon Postel's [robustness principle](https://en.wikipedia.org/wiki/Robustness_principle) is a
fixture of software design and implementation advice. It states:

> Be liberal in what you accept, and conservative in what you send.

In recent years, this counsel has [proven controversial][postel-was-wrong] and for a number of
reasons, this handbook discourages broad application of the principle. This is a reversal of
[previous guidance][mea-culpa].

[postel-was-wrong]: https://datatracker.ietf.org/doc/html/draft-thomson-postel-was-wrong-03
[mea-culpa]: https://github.com/ibm-cloud-docs/api-handbook/blob/086d28c9357b39612ceb816130d0f6ad92f82859/design/errors.md#robustness-tradeoffs

### Historical meaning

There are many possible applications of the robustness principle, and its original context (as
[highlighted][fallibility-of-specifications] by Martin Thomson's dissent) is not applicable to
new API design:

> While the goal of this specification is to be explicit about the protocol there is the
> possibility of differing interpretations. In general, an implementation should be conservative in
> its sending behavior, and liberal in its receiving behavior.

Paraphrased, the narrower, original definition of the robustness principle states that if a
protocol's specification exists apart from sundry implementations, a defensive implementor should
react to ambiguity (which is a _defect_ of the specification) with:

 * A broad interpretation of what constitutes valid input (but still only within the confines of what the specification leaves open to interpretation) 
 * A narrow interpretation of what constitutes valid output

[fallibility-of-specifications]: https://datatracker.ietf.org/doc/html/draft-thomson-postel-was-wrong-03#section-2

## Best practices
{: #best-practices }

When a service team is responsible for developing an API definition and its canonical
implementation (the _service_) in concert, Postel's robustness principle SHOULD NOT be
(mis)applied. This section details best practices that contradict adherence to the
robustness principle as it is colloquially understood.

### Specificity of validity

The formal definition and documentation for a service API SHOULD be as explicit as possible about
what constitutes valid input.

A client developer shouldn't have to guess at the boundaries of valid input. Armed with explicit
knowledge of input validity, developers can write client applications which are _more_ resistant to
unexpected failure (and, in this way, are more robust). This specificity also allows quality and
security testing to employ [boundary-value analysis][boundary-testing].

A service implementation SHOULD NOT be liberal in accepting input where the API definition is
ambiguous about validity. Instead, the definition SHOULD be updated to remove such ambiguity.

[boundary-testing]: https://en.wikipedia.org/wiki/Boundary-value_analysis

### Input canonicalization
{: #input-canonicalization}

The formal definition for a service API SHOULD NOT allow a broad range of semantically equivalent
inputs without strong justification or mitigation of the inherent problems.

There are usability benefits to accepting various forms of input. It can be more convenient
for a developer — and lower an API's barrier to entry — if a client application doesn't have to
canonicalize input before sending it.

But the downsides of accepting noncanonical input can be significant:

* Canonicalization can be a complicated procedure with many corner cases and ambiguities that can
  significantly increase the complexity of an API. This additional complexity makes the service
  harder to write and maintain and increases the testing and attack surfaces.
* If canonicalization decisions are tied to specific libraries used by the service, it may be
  significantly harder to transition a service implementation to a new technology.
* If canonicalization is done lazily[^lazy-canonicalization], the burden falls on clients to
  compute equivalency of values, and regardless of how noncanonical values are sorted, some client
  developers may be [surprised][astonishment].
* If canonicalization is done eagerly[^eager-canonicalization], some clients — and in particular,
  [declarative orchestration engines][infrastructure-as-code] — may experience undesired behavior
  if a value the client sets is silently transformed such that the client may incorrectly assume
  that a value it set was overridden (by another client in a race), causing the client to repeatedly
  attempt to reset the value.
* Besides increasing overall attack surface, certain classes of attacks specifically depend on
  bugs where different code paths disagree on canonicalization logic.

[^lazy-canonicalization]: That is, noncanonical values are accepted, stored, and returned,
  and only canonicalized (opaquely to the user) when necessary.

[^eager-canonicalization]: That is, noncanonical values are canonicalized before being stored
  and returned only in canonical form.

[astonishment]: https://en.wikipedia.org/wiki/Principle_of_least_astonishment
[infrastructure-as-code]: https://en.wikipedia.org/wiki/Infrastructure_as_code

#### Exceptions

If an industry-standard format used has noncanonical forms that a client developer would reasonably
expect to be able to send, and canonicalization is well-documented and widely understood, eager
canonicalization SHOULD be used.

For example, [RFC 3339 date-times values][date-time-inputs] with timezone offsets SHOULD be
canonicalized and returned in UTC. The ubiquitous support for such canonicalization mitigates the
downsides, and together with a developer's expectation that a full industry-standard format would
be supported, the benefits exceed the risks in such a case.

[date-time-inputs]: /docs/api-handbook?topic=api-handbook-types#date-time-accepted-formats

### Sanitation and validation
{: #sanitation-and-validation}

Definitively invalid or ambiguous input MUST NOT be ignored and therefore MUST result in an error.
_Validation_ is better than _sanitation_.

There are several reasons ignoring invalid input is dangerous and ill-advised. The following are
hazards of a service that ignores invalid input:

* Client developers may be confused as to why a feature is not working as expected, when in reality
  their request is inadvertently malformed in a way that's hard to spot.
* Subtle bugs in client applications may result in dangerous unintentional consequences. For
  example, if a filter parameter on a list operation is ignored because a malformed value is
  unintentionally provided, a client might misconfigure or delete the wrong set of resources. In
  extreme cases such bugs could result in outages of dependent services, loss of data, or accidental
  access grants to resources.
* Subtle bugs in service code that fail to fully discard invalid input in all contexts
  could result in attack vectors.

### Extraneous input

Except where required by an industry standard or convention[^ignore-unknown-headers], extraneous
input, such as unrecognized properties in JSON request bodies and unrecognized query parameters,
SHOULD NOT be ignored and SHOULD result in an error.

[^ignore-unknown-headers]: Such as RFC 7230's 
  [requirement](https://datatracker.ietf.org/doc/html/rfc7230#section-3.2.1) that unknown HTTP
  headers be ignored.
  
All the downsides listed for [silently ignoring invalid input](sanitation-and-validation) also
apply to ignoring extraneous input.

Additionally, ignoring extraneous input causes particular hazards for evolving a service in a
backward-compatible way. Specifically, if a client is sending a property or parameter that is
unsupported _and ignored_, then the client application may experience a failure or an outage if and
when support is added to the service for a property or parameter of the same name.

## Migrating to best practices for robustness
{: #migrating-to-best-practices }

Migrating a service API that is less strict in its handling of requests to comply with the [best
practices](#best-practices) for robustness inevitably involves making [backward-incompatible
changes][robustness-changes].

When feasible, existing, noncompliant service APIs SHOULD be updated to adhere to best practices
by using one of the following approaches:

* If a service API has a small, known set of consumers, and log analysis can be used to identify
  client behavior that would run afoul of stricter validation, white-glove assistance to client
  developers MAY be offered ahead of changes that would otherwise be considered breaking.
* If a service API employs [date-based versioning][date-based-versioning], stricter enforcement MAY
  be added for a new version date. If feasible, updated enforcement SHOULD happen in a single
  version date across the whole service. For large services, an iterative approach where cohesive
  sections of the API are updated in a series of versioned changes MAY be used.
* If a new major version for a service API is developed, updated robustness best practices MUST be
  enforced across the new version.
  
For existing service APIs, new operations and new request headers, request schema properties, and
query parameters on existing operations SHOULD adhere to updated robustness best practices with
respect to invalid and extraneous input but MAY maintain consistency with existing API features
with respect to canonicalization.

[robustness-changes]: /docs/api-handbook?topic=api-handbook-change-compatibility#robustness-changes
[date-based-versioning]: /docs/api-handbook?topic=api-handbook-changes-overview

## Optimistic locking

In cases where races between clients could result in unintended resource configurations, optimistic
locking through [validator][validator-headers] and [conditional][conditional-headers] headers SHOULD be
supported.

In particular, the following operations SHOULD support optimistic locking:

* A `PUT` operation that wholly replaces an existing resource
* A `PATCH` operation that supports JSON merge-patch and the mutation of an array field

If specific kinds of requests are deemed particularly dangerous in race conditions, a service MAY
require that the client provide an `If-Match` header.

[validator-headers]: /docs/api-handbook?topic=api-handbook-headers#validator-headers
[conditional-headers]: /docs/api-handbook?topic=api-handbook-headers#conditional-headers
