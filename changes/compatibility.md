---

copyright:
  years: 2020
lastupdated: "2020-02-13"

subcollection: api-handbook

---

{:important: .important}

# Change compatibility

API changes must be assessed and designed to consider compatibility. The backward-compatibility of
each change MUST be determined by using the directives in the [determining
compatibility](#determining-compatibility) section.

Backward-compatible changes SHOULD be made as **evolutionary** updates and backward-incompatible
changes SHOULD be made as **versioned** updates. Rarely, backward-incompatible changes MAY be made
as **breaking** updates.

#### Evolutionary updates

Evolutionary updates incorporate backward-compatible changes into an existing major version of an
API and (if applicable) across all version dates. Client applications need not use a new major
version or new version date to employ features made available in such updates.

If feasible without significant compromise to the usability, consistency, or robustness of the API,
a change SHOULD be designed for backward-compatibility and made as an evolutionary update.

#### Versioned updates

Versioned updates incorporate backward-incompatible changes into a new major version or a new
version date. Versioned updates require client applications to specify the new version to use
features made available in the changes, and protect unchanged clients from being impacted by the
updates.

If a change cannot be made backward-compatible, or if backward-incompatibility is necessary to
maintain the usability, security, or robustness of the API, the change SHOULD be made as a versioned
update.

#### Breaking updates

In rare cases, it might be necessary to make a backward-incompatible change retroactive for a major
version and (if applicable) across all version dates.

Exceptions for breaking updates MUST be justified by a security concern, legal obligation, or
significant cost of continuing to support the previous behavior.

## Determining compatibility

No universal criterion exists for what kind of changes could disrupt client applications. Whether a
change could disrupt a specific client application depends on what assumptions the individual client
developer made. For this reason, any change must be carefully evaluated for what reasonable
assumptions which could have been made by client developers would render the change incompatible
with existing code.

The following directives MUST be used as the basis for classifying changes but SHOULD be
supplemented with analysis of real-world API usage.

### Backward-compatible changes

The following kinds of changes SHOULD be considered backward-compatible and supported in
evolutionary updates:

- Adding a new path
- Adding a new method to an existing path
- Adding a new property to a response schema
- Supporting a new optional query parameter, header, or property in a request schema
- Expanding acceptable values for an existing query parameter, header, or property in a request
  schema
- Reducing possible values for a property in a response schema

#### Special cases for backward-compatibility

The following is provisional guidance and will be updated in an upcoming revision of the API
Handbook.
{: important}

Some other kinds of changes MAY be considered backward-compatible under certain conditions and
supported in evolutionary updates. Such changes can be considered compatible only if they are
designed to avoid impacting existing client applications that take either of the following client
robustness approaches across an entire resource scope[^resource-scope]:

1. **Isolation-based client robustness:** Client developers ensure that no use is made of new
   features[^new-features] by any client applications across a resource scope. Existing client
   applications will not encounter any effects of the changes because these applications will
   operate only on resource configurations which were supported prior to the changes being made
   available.

   In support of this approach, the service MUST ensure that resources taking advantage of expanded
   response schemas will not appear in a resource scope where a client application did not
   explicitly request them.

2. **Forward-compatible client robustness:** Client developers ensure that all client applications
   across a resource scope are coded to gracefully handle new behavior (such as property values
   outside previously documented ranges), in a _forward-compatible_ way, as advised in the API
   documentation.

   In support of this approach, the API documentation MUST clearly describe the potential for new
   behavior and offer guidance on handling future response schema expansions. This guidance must
   have been in place when the functionality being changed was originally documented.

The following types of changes are examples of what MAY be considered backward-compatible if the
preceding client robustness approaches can be respected:

- Expanding possible values for a property in a response schema; for example, a new value in an
  enumeration, a lower minimum or higher maximum, or a more permissive string pattern.
- Adding a new required property to a previously unsupported variant of a request
  schema[^new-required-property].
- Reducing acceptable values for a property for a previously unsupported variant of a request
  schema[^reducing-acceptable-values].
- Expanding the types of resources that could be referenced by an `href` or embedded reference
  schema within the context of another resource.

#### Changing default values

Additionally, a default value for a property in a request MAY be changed if the new default has no
increase in price or regressions in performance or supported features at the time of the change.

For example, the default pricing profile for a new resource may be changed if the new default is
considered better in all respects and is an equal or lower price at the time of the change. (The
profile that was previously default may be made less expensive at the same time, or after, but not
before the change.)

#### Resolving API definition errata

If the implementation for an API is well-designed and functional, but the API definition differs
from the implementation in an incompatible way, the definition SHOULD be updated to match the
implementation, despite the resultant appearance of an incompatible change.

### Backward-incompatible changes

A change that meaningfully alters the outcome or response format of any currently valid request MUST
be considered backward-incompatible. Backward-incompatible changes SHOULD be supported in versioned
updates.

The following kinds of changes SHOULD be considered backward-incompatible unless [a special
case](#special-cases-for-backward-compatibility) applies:

- Removing or renaming an existing path
- Removing support for an existing method on an existing path
- Removing or renaming an existing query parameter
- Removing or renaming an existing property in a request or response schema
- Adding a new required query parameter, header, or property in a request schema
- Changing the status code for a particular scenario (except when the existing status code is `404`)
- Reducing acceptable values for an existing query parameter, header, or property in a request
  schema
- Changing the semantics of a value for an existing query parameter, header, or property in a
  response schema
- Changing a default value or behavior for an already-valid request
- Redefining the nature of any relationship between resources

[^resource-scope]: Where a "resource scope" is the enclosing environment for resources, such as a
specific account on a specific service deployment.

[^new-features]: New features made available by the API changes.

[^new-required-property]: That is, an existing request schema may only have a newly required
  property in combination with values for the rest of the schema's properties that would have
  previously been invalid.

[^reducing-acceptable-values]: In other words, acceptable values for an existing property must only
  be eliminated in combination with values for the rest of the schema's properties that would have
  previously been invalid.
