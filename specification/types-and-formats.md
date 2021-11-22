---

copyright:
  years: 2019
lastupdated: "2019-11-06"

subcollection: api-handbook

---

# Types and formats
{: #types-and-formats}

Property and parameter schema MUST only use combinations of `type` and `format` defined in OpenAPI
Specification
[ref](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md#data-types), with
the following additions:

| `type`   | `format`     | Comments |
| -------- | ------------ | -------- |
| `string` | `identifier` | identifier for a resource (see [Identifier in Types](/docs/api-handbook/design/types#identifier)) |
| `string` | `crn`        | the Cloud Resource Name for a resource (see [CRN in Types](/docs/api-handbook/design/types#crn)) |
{: caption="Types and formats" caption-side="bottom"}

## Specifying additional constraints
{: #specifying-additional-constraints}

Some constraints needed for input validation or output guarantees are not naturally expressed in
OpenAPI. The following are recommendations for documenting such constraints.

## Date-time precision
{: #date-time-precision}

To document the precision of `date-time` properties in responses, use the following additional
constraints:

### For second precision:
{: #for-second-precision}

```yaml
minLength: 20
maxLength: 20
```

### For millisecond precision:
{: #for-millisecond-precision}

```yaml
minLength: 24
maxLength: 24
```

## String character sets
{: #string-character-sets}

Where practical, constrain the character set for string properties by using the schema's `pattern`.
Regardless, describe the character set constraints in plain language in the schema's description.
