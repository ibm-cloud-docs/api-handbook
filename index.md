---

copyright:
  years: 2019, 2021
lastupdated: "2021-07-06"

subcollection: api-handbook

keywords: "API,API best practices,API Handbook"

---

# Introduction
{: #intro}

This handbook is designed to serve as a set of standards, styles, and best practices for IBM Cloud
APIs. It is expected to grow and evolve over time in order to best address the needs and
requirements of users and developers of IBM Cloud APIs. To learn how to get involved, see
[Contributing](/docs/api-handbook?topic=api-handbook-contributing).

## Using this handbook

This handbook is intended to be used by IBM Cloud service architects and developers as guidelines
for designing any REST API exposed publicly by IBM Cloud services. Private or internal APIs SHOULD
also follow these guidelines for providing the same level of consistency to internal service
consumers and simplify the process if the internal APIs are subsequently exposed publicly.

There are legitimate reasons for a service to be exempted from adhering to this handbook's
guidelines. For example, an exemption may be approved if a service offers an API compatible with a
de facto standard, such as S3, or if a service offers an API that seamlessly extends an open
source project, such as Kubernetes, following that project's API standards and conventions.
{:tip}

## Conventions used

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC
2119](https://tools.ietf.org/html/rfc2119).

## Verifying compliance

The [OpenAPI Validator tool](https://github.com/IBM/openapi-validator) validates OpenAPI documents
against a subset of the standards defined in this handbook and identifies areas of non-compliance with
the OpenAPI specification or the guidelines in this handbook.

The output of the OpenAPI Validator is a report that identifies specific elements of the API definition
that fail to comply with the OpenAPI specification or the IBM API Handbook. All errors listed by the
report MUST be resolved prior to publication of the API unless the remediations would result in
breaking changes for an existing API version.

Resolving all issues reported by the validator does not guarantee full compliance
with this handbook's standards. API designers MUST perform further manual validations for the guidelines
that cannot be checked automatically by the tool.
{:note}
