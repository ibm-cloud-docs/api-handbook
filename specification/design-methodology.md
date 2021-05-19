---

copyright:
  years: 2021
lastupdated: "2021-05-17"

subcollection: api-handbook

---

# Design methodology

## Design-first API definitions

Service teams SHOULD employ a design-first approach to authoring API definitions. In particular, an
API definition SHOULD be authored deliberately as source of truth for the API contract, and not as a
byproduct of annotated service code.

Careful and intentional authorship of an API definition is important, because that definition will
likely be used:

- To generate API reference documentation for client developers
- To generate native, type-aware client SDKs
- To develop or generate UIs, CLIs, or adaptors for orchestration engines
- To develop, generate, or conduct tests
- To evaluate or probe a service's attack surface
- To understand how a service can evolve in a backward-compatible way

An API definition MAY also be used to generate models (such as structs or classes), validators, or
handler stubs for service code. Unlike generating an API definition from service code, this practice
is encouraged.

API definitions are an important point of convergence to drive understanding and consensus among
stakeholders for any project or product with an API. At every stage of authorship, from early draft
to committed contract, an API definition SHOULD be accessible to all stakeholders, such as product
managers, architects, back-end engineers, front-end engineers, quality engineers, designers, and
technical writers.

## API definition management

An API definition MUST be managed under source control. An API definition MAY be source controlled
in the exact form of its publication or MAY use a multi-file arrangement or other preprocessing for
redaction, deduplication, versioning, or internal annotations. An API definition managed in a form
that requires preprocessing before publication MUST use an automated build to keep resulting artifacts
up to date.

The source repository for an API definition MUST have a specific branch or build artifact
that represents the current source of truth for the service's latest-version production API behavior.
Other branches or build artifacts MAY represent other versions or agreed-upon future API updates.
Updates to an API definition MUST require a code review and SHOULD be contingent on overall
architectural review for service updates.

The [IBM OpenAPI validator][validator] SHOULD be used for automatic verification of every update to
an API definition (or every resulting build artifact).

[validator]: https://github.com/IBM/openapi-validator
