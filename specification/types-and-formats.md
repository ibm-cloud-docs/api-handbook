---

copyright:
  years: 2019
lastupdated: "2019-11-06"

subcollection: api-handbook

---

# Types and formats

Property and parameter schema MUST only use combinations of `type` and `format` defined in OpenAPI
Specification
[ref](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md#data-types), with
the following additions:

`type` | `format` | Comments
------ | -------- | --------
`string` | `identifier` | identifier for a resource (see [Identifier in Types](/docs/api-handbook/design/types#identifier))
`string` | `crn` | the Cloud Resource Name for a resource (see [CRN in Types](/docs/api-handbook/design/types#crn))
