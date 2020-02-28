---

copyright:
  years: 2019
lastupdated: "2019-12-01"

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
also follow these guidelines for providing the same level of consistency also to internal service
consumers and simplify the process if eventually the internal APIs will be exposed publicly.

There are legitimate reasons for exemption from implementing this guidelines. For example in case
the service need to adhere to well-established industry standards (S3, OpenStack, etc.), must
interoperate with some externally defined REST API (Kubernetes CRD), etc.

## Conventions used

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC
2119](https://tools.ietf.org/html/rfc2119).

## Verifying compliance

The [OpenAPI Validator tool](https://github.com/IBM/openapi-validator) validates OpenAPI documents
against a subset of the  standard defined in this handbook and identify areas of non-compliance with
the OpenAPI specification or the IBM API Guidelines.

The output of the OpenAPI Validator is a report identifying specific elements of the API definition
that fail to comply with the OpenAPI specification or the IBM API Handbook. All errors listed by the
report MUST be resolved prior to publication of the API unless, as described in the previous
section, the remediations create major breaking changes with previous version of the API.

Even resolving all the issues reported by the validator it will not guarantee full compliance of
with the handbook standard. API designer MUST perform further manual validations for the guidelines
that cannot be checked automatically by the tool.
