---

copyright:
  years: 2022
lastupdated: "2022-03-21"

subcollection: api-handbook

---

{:external: .external}

# Writing
{: #writing}

## Writing style
{: #writing-style}

Written content in an API definition, such as `summary` and `description` properties, MUST contain
American English. Content SHOULD be written for client developers as the primary audience. A
particular type of client, such as an interactive utility, a dashboard, or scheduled automation
SHOULD NOT be assumed.

All content SHOULD use sentence-style capitalization.

### Programmatic and conceptual values
{: #programmatic-and-conceptual}

When referring to a programmatic value such as a property name, boolean, or enumerated value, the
value SHOULD be written precisely as in code, and surrounded by backticks to indicate inline code in
[CommonMark][commonmark-help]{: external}. For example:

*  "Specify the `user_id` property..."
*  "Set `debug` to `true`..."
*  "In the `2021-12-31` API version..."

When referring to the concept that the syntax of an API represents, use ordinary text, even if a
programmatic value has the same name as the concept. For example:

*  "The password must be at least 12 characters in length."
*  "The user ID of an administrator..."
*  "Use debug mode to see additional messages."

[commonmark-help]: https://commonmark.org/help/

## Operations
{: #operations}

### Operation summaries
{: #operation-summaries}

The `summary` value for an operation SHOULD be parallel to the [operation ID][operation-ids], in
sentence case, without punctuation, and with an indefinite article where appropriate. A summary
SHOULD be as short as possible while remaining unique and unambiguous.

[operation-ids]: /docs/api-handbook?topic=api-handbook-operations#operation-ids

For example:

*  For `list_users`, the summary would be `List users`
*  For `create_user`, the summary would be `Create a user`
*  For `get_user`, the summary would be `Get a user`

A summary MAY omit parts of the resource hierarchy as long as the resulting text remains clear. For
example the `get_farm_barn` (`GET /farms/{farm_id}/barns/{id}`) operation could have the summary
`Get barn`.

A summary MUST NOT contain CommonMark formatting.

### Operation descriptions
{: #operation-descriptions}

The `description` value for an operation SHOULD be in full, grammatically correct sentences. The
description SHOULD expand on the operation summary (without depending on the summary for
context) and include any important details about the purpose and scope of the operation.

An operation description SHOULD also elaborate any important operation-spanning constraints or
behaviors that cannot be clearly explained in property or parameter descriptions.

An operation description MAY contain paragraphs, lists, links, and other CommonMark formatting.

## Property and parameter descriptions
{: #property-and-parameter-descriptions}

The `description` value for a property or parameter SHOULD start with a sentence-fragment
definition, ending with a period. For example, "A network-connected block storage device."

These descriptions SHOULD also elaborate on the purpose and usefulness of the property or parameter
along with any constraints or behaviors that cannot be defined with native OpenAPI schema keywords,
as recommended for various [Types][types].

[types]: /docs/api-handbook?topic=api-handbook-types

A property or parameter description MAY contain paragraphs, lists, links, and other CommonMark
formatting.
