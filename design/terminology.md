---

copyright:
  years: 2019, 2025
lastupdated: "2025-03-28"

subcollection: api-handbook

---
# Terminology
{: #terminology}

This section applies to the names of model field names, URL segment names, query parameter names,
and enumeration value names. In this document, these are collectively referred to as "names."

## Construction

Names for resources and field names SHOULD be chosen based on the nature, purpose, and role of the
resource or data encapsulated. A resource name SHOULD end with a noun and a field name SHOULD end
with a noun or a preposition.

Field names MUST NOT repeat the type of the resource (for example, fields in a `Widget` resource
MUST NOT be prefixed with `widget_`).

Names MUST NOT include generic, semantically void terms such as `info` or `details`. For example,
rather than encapsulating `endpoint` and `credentials` fields in a model within a field called
`database_info`[^database-info], use `database_connection` which accurately describes the nature of
the modeled state.

[^database-info]: Where `database` isn't satisfactory because the credentials do not describe the
   database per se and `database_credentials` isn't satisfactory because the endpoint is not a
   part of the credentials.

## Formatting
{: #formatting}

Names MUST be lower snake case. This format is defined as one or more lowercase words separated by
the underscore character (`_`). These names MUST NOT begin with an underscore or number.

## Abbreviations
{: #abbreviations}

Names MUST NOT contain abbreviations which are not industry-standard. Where industry-standard
abbreviations are used, they MUST be used consistently.

## Brand names
{: #brand-names}

Brand names and code names SHOULD be avoided in field names, URL segment names, and query parameter
names. Brand names MAY be used in enumeration value names, if necessary. Where possible, resources
and resource fields SHOULD be named with industry-standard, widely-understood terms.

## Language
{: #language}
Names MUST be in American English.
