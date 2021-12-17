---

copyright:
  years: 2019, 2021
lastupdated: "2021-06-30"

subcollection: api-handbook

---

{:external: .external}
{:important: .important}
{:note: .note}

# Types
{: #types}

Types define particular kinds of values. Each field in a model MUST have exactly one type, and
fields of some types cannot be optional, mutable, or (even in JSON merge patch requests) nullable.
Types also govern how a value from (or for) a particular field can be represented in JSON and in
query parameters.

The following types are described below:

*  [Identifier](#identifier)
*  [Boolean](#boolean)
*  [Integer](#integer)
*  [Float](#float)
*  [String](#string)
*  [Date](#date)
*  [Date/Time](#datetime)
*  [CRN](#crn)
*  [Enumeration](#enumeration)
*  [Model](#model)
*  [Array](#array)

## Identifier
{: #identifier}

The identifier type contains a value which identifies a resource. It MUST uniquely identify a
resource among all resources of the same type, and SHOULD uniquely identify a resource among all
resources in a system. Identifier values MAY be [RFC
4122](https://datatracker.ietf.org/doc/html/rfc4122){: external} UUIDs and SHOULD be selected by the
system to ensure uniqueness.

It is not required that all models have identifier values. However, a model which has an identifier
value MUST include it in a self-referential field named `id`. Fields other than `id` MAY have the
identifier type, but such fields MUST refer to other resources.

### Response format
{: #identifier-return-format}

An identifier value MUST be returned as a JSON string. Case-insensitive identifiers MUST be returned
in consistent case and SHOULD be returned in lowercase.

For responses, an identifier field SHOULD have a documented maximum length and character set. For
the sake of backward-compatible future expansions, documented constraints MAY be more permissive
(allowing for a longer value or larger character set) for responses than for requests.

A resource's self-referential `id` field MUST be required in any object schema used in a response
and representing the resource. An identifier field that expresses an optional binding to a related
resource MAY be optional.

| Constraint  | Recommended value | Description |
| ----------- | ----------------- | ----------- |
| `type`      | `string`          | `type` MUST be `string`. |
| `format`    | `identifier`      | `format` MUST be `identifier`. |
| `maxLength` | `128`             | `maxLength` SHOULD be present. |
| `pattern`   | `^[-0-9a-z]+$`    | `pattern` SHOULD be present. |
{: caption="Schema guidance for identifiers in responses" caption-side="bottom"}

### Request formats
{: #identifier-accepted-formats}

For requests, an identifier field MUST have a documented maximum length and SHOULD have a maximum
length of 128 characters or fewer. An identifier MUST have a permissible character set limited to
printable ASCII and SHOULD have a permissible character set further constrained to alphanumeric
characters and the hyphen (`-`).

Length and character set constraints MUST be checked prior to any other processing (including case
normalization[^identifier-case-normalization]) of an identifier included in a request. For fields
containing system-generated identifiers, documented constraints MAY be more permissive (allowing
for a longer value or larger character set) than the implementation permits. Failure to match
constraints MUST cause the entire request to fail with a `400` status code and appropriate error
response model.

[^identifier-case-normalization]: Occasionally, case normalization may coerce a non-ASCII character
   into an ASCII character. This MUST NOT be allowed to occur.

In request bodies and query parameters, identifier values SHOULD be matched case-insensitively.

A resource's self-referential `id` field MUST be excluded[^exclude-id-from-request] from any object
schema used in a request to mutate that resource.

[^exclude-id-from-request]: By omission or by marking it as read-only.

| Constraint  | Recommended value    | Description |
| ----------- | -------------------- | ----------- |
| `type`      | `string`             | `type` MUST be `string`. |
| `format`    | `identifier`         | `format` MUST be `identifier`. |
| `maxLength` | `128`                | `maxLength` MUST be present. |
| `pattern`   | `^[-0-9A-Za-z]+$`    | `pattern` MUST be present. |
{: caption="Schema guidance for identifiers in requests" caption-side="bottom"}


## Boolean
{: #boolean}

The boolean type contains either a `true` or `false` value.

### Response format
{: #boolean-return-format}

A boolean value MUST be returned as the JSON keyword `true` or the JSON keyword
`false`.[^json-keywords]

In a response body, a boolean field MUST be required. If the property is inapplicable to certain
states of the resource, this ternary capability MUST be represented explicitly with a enumeration
containing values such as `on`, `off`, and `inapplicable`.

| Constraint  | Recommended value    | Description |
| ----------- | -------------------- | ----------- |
| `type`      | `boolean`            | `type` MUST be `boolean`. |
{: caption="Schema guidance for booleans in responses" caption-side="bottom"}

### Request formats
{: #boolean-accepted-formats}

#### Request bodies
{: #boolean-request-bodies}

*  The JSON keyword `true` or the JSON keyword `false` MUST be considered a valid boolean.
*  Any other value MUST be rejected with a `400` status code and appropriate error response model.

In a request body, a boolean field MAY be optional. An optional boolean field MUST have either an
explicit default value or an explanation of the strategy used to compute a value in the property
schema `description`.

#### Query parameters
{: #boolean-query-parameters}

*  The strings `true` and `false` MUST be case-insensitively matched as valid booleans.
*  Any other string MUST be considered invalid and SHOULD be rejected with a `400` status code and
   appropriate error response model.

The character set of a boolean query parameter value within a request MUST be verified prior to case
normalization[^boolean-case-normalization].

Despite all values being encoded as strings within the request URL, a boolean query parameter MUST
be defined with a `boolean` schema. OpenAPI does not itself specify valid encoding for query
parameters with `boolean` schemas, which makes the preceding guidance crucial.
{: note}

[^boolean-case-normalization]: Occasionally, case normalization may coerce a non-ASCII character
   into an ASCII character. This MUST NOT be allowed to occur.

| Constraint  | Recommended value    | Description |
| ----------- | -------------------- | ----------- |
| `type`      | `boolean`            | `type` MUST be `boolean`. |
{: caption="Schema guidance for booleans in requests" caption-side="bottom"}


## Integer
{: #integer}

The integer type provides for whole number values between `-2^53 + 1` (-9,007,199,254,740,991) and
`2^53 - 1` (9,007,199,254,740,991)[^min-max-int].

[^min-max-int]: The JSON specification itself [defers to implementing
   software](https://datatracker.ietf.org/doc/html/rfc7159#section-6){: external} on maximum safe
   sizes for integers but suggests that guaranteed precision cannot be assumed to be interoperable
   for values outside of the safe integer range for IEEE 754 double-precision values.

### Response format
{: #integer-return-format}

In a response body, an integer field SHOULD have documented minimum and maximum values. For the
sake of backward-compatible future expansions, documented constraints MAY be more permissive
(allowing for a greater range of values) for responses than for requests.

An integer value MUST be returned as a JSON number and MUST NOT have a decimal point or use
scientific notation.

| Constraint  | Recommended value                                              | Description |
| ----------- | -------------------------------------------------------------- | ----------- |
| `type`      | `integer`                                                      | `type` MUST be `string`. |
| `format`    | `int32` or `int64`                                             | `format` MUST be `int32` or `int64`. |
| `minimum`   | ≥`-2147483648` for `int32` or ≥`-9007199254740991` for `int64` | `minimum` value SHOULD be present. |
| `maximum`   | ≤`2147483647` for `int32` or ≤`9007199254740991` for `int64`   | `maximum` value SHOULD be present. |
{: caption="Schema guidance for integers in responses" caption-side="bottom"}

### Request formats
{: #integer-accepted-formats}

In a request body, an integer field MUST have documented minimum and maximum values.

Minimum and maximum constraints MUST be checked prior to any other processing of an integer
included in a request. Failure to match constraints MUST cause the entire request to fail with a
`400` status code and appropriate error response model.

#### Request bodies
{: #integer-request-bodies}

*  Any JSON number value which evaluates to an integer within the service providing an API MUST be
   considered a valid integer, but MAY be rejected by other validation rules.
*  The `null` JSON keyword MAY be accepted in a JSON merge patch request body if the field is
   optional in the resource state and MUST NOT be accepted otherwise.
*  Any other value MUST be rejected with a `400` status code and appropriate error response model.

#### Query parameters
{: #integer-query-parameters}

*  Any string containing a [valid JSON
   number](https://datatracker.ietf.org/doc/html/rfc7159#section-6){: external} value without a fractional or
   exponential value MUST be considered a valid integer.
*  Any other string containing a valid JSON number (that is, a string with a fractional or
   exponential value) MAY be used for comparative filtering but MUST NOT be considered an exact
   match.
*  The lowercase string `null` MUST be considered a match for a value absent from the observable
   resource state.
*  Any other string MUST be considered invalid and SHOULD be rejected with a `400` status code and
   appropriate error response model.

| Constraint  | Recommended value                                              | Description |
| ----------- | -------------------------------------------------------------- | ----------- |
| `type`      | `integer`                                                      | `type` MUST be `string`. |
| `format`    | `int32` or `int64`                                             | `format` MUST be `int32` or `int64`. |
| `minimum`   | ≥`-2147483648` for `int32` or ≥`-9007199254740991` for `int64` | `minimum` value MUST be present. |
| `maximum`   | ≤`2147483647` for `int32` or ≤`9007199254740991` for `int64`   | `maximum` value MUST be present. |
{: caption="Schema guidance for integers in requests" caption-side="bottom"}


## Float
{: #float}

The float type provides for numeric values which cannot be represented as integers and for which
guaranteed precision is not required. A float field MUST NOT purport to support values or precision
beyond a 64-bit `double`[^64-bit-double]

[^64-bit-double]: The JSON specification itself [defers to implementing
   software](https://datatracker.ietf.org/doc/html/rfc7159#section-6){: external} on maximum sizes
   and precision guarantees for numbers. For interoperability, this handbook requires that no
   service expect or return floating-point values outside of what can be stored as a 64-bit
   `double`.

### Response format
{: #float-return-format}

For responses, a float field SHOULD be documented as either a 32-bit `float` or a 64-bit `double`
and MAY further constrain the range of valid values. For the sake of backward-compatible future
expansions, documented constraints MAY be more permissive (allowing for a larger range of values)
for responses than for requests.

A float value MUST be returned as a JSON number and MAY have a decimal point or use scientific
notation.

| Constraint  | Recommended value   | Description |
| ----------- | ------------------- | ----------- |
| `type`      | `number`            | `type` MUST be `number`. |
| `format`    | `float` or `double` | `format` MUST be `float` or `double`. |
| `minimum`   |                     | `minimum` value MAY be present. |
| `maximum`   |                     | `maximum` value MAY be present. |
{: caption="Schema guidance for floats in responses" caption-side="bottom"}

### Request formats
{: #float-accepted-formats}

For requests, a float field MUST document and enforce support for either a 32-bit `float` or a
64-bit `double` and MAY further constrain the range of valid values.

Minimum and maximum constraints MUST be checked prior to any other processing of a float included in
a request. Failure to match constraints MUST cause the entire request to fail with a `400` status
code and appropriate error response model.

#### Request bodies
{: #float-request-bodies}

*  Any JSON number value MUST be considered a valid float, but MAY be rejected by other validation
   rules.
*  The `null` JSON keyword MAY be accepted in a JSON merge patch request body if the field is
   optional in the resource state and MUST NOT be accepted otherwise.
*  Any other value MUST be rejected with a `400` status code and appropriate error response model.

#### Query parameters
{: #float-query-parameters}

*  Any string containing a [valid JSON
   number](https://datatracker.ietf.org/doc/html/rfc7159#section-6){: external} value MUST be considered a
   valid float.
*  The lowercase string `null` MUST be considered a match for a value absent from the observable
   resource state.
*  Any other string MUST be considered invalid and SHOULD be rejected with a `400` status code and
   appropriate error response model.

| Constraint  | Recommended value   | Description |
| ----------- | ------------------- | ----------- |
| `type`      | `number`            | `type` MUST be `number`. |
| `format`    | `float` or `double` | `format` MUST be `float` or `double`. |
| `minimum`   |                     | `minimum` value MAY be present. |
| `maximum`   |                     | `maximum` value MAY be present. |
{: caption="Schema guidance for floats in requests" caption-side="bottom"}


## String
{: #string}

The string type provides for arbitrary strings of characters. Note that the string type referred to
here is not equivalent to a JSON string, as JSON strings are also used to represent dates,
date/times, and enumerations.

### Response format
{: #string-return-format}

For responses, a string field SHOULD have a documented maximum length and SHOULD have a documented
permissible character set. For the sake of backward-compatible future expansions, documented
constraints MAY be more permissive (allowing for longer strings or a larger character set) for
responses than for requests.

A string value MUST be returned as a JSON string.

A free-form string field (such as a `description` or `notes` field) that allows a zero-length value
MUST NOT be optional in a response body. A string field set to an empty string MUST be represented
as a literal empty string (`""`).

| Constraint  | Recommended value | Description |
| ----------- | ----------------- | ----------- |
| `type`      | `string`          | `type` MUST be `string`. |
| `minLength` |                   | `minLength` SHOULD be present. |
| `maxLength` |                   | `maxLength` SHOULD be present. |
| `pattern`   |                   | `pattern` SHOULD be present unless character constraints cannot be expressed in a regular expression. |
{: caption="Schema guidance for strings in responses" caption-side="bottom"}

### Request formats
{: #string-accepted-formats}

For requests, a string field MUST have documented minimum and maximum lengths and MUST have a
documented permissible character set. A string field that does not contain completely free-form
values SHOULD have a documented regular expression pattern which values are required to match.

Length, character set, and pattern constraints MUST be checked prior to any other processing of
strings included in requests. Failure to match constraints MUST cause the entire request to fail
with a `400` status code and appropriate error response model.

A free-form string field (such as a `description` or `notes` field) that allows a zero-length value
SHOULD be optional in a request body, and if it is optional, its default value MUST be a literal
empty string (`""`).

A zero-length string value MUST NOT be accepted to use a default or computed value or to omit a
field entirely from observable resource state. For example, if a request has an optional
`public_ip_address` field and omitting or providing a `null` value (in a JSON merge patch request)
for the field results in a resource without a public IP address, providing a zero-length value MUST
NOT do the same.

| Constraint  | Recommended value    | Description |
| ----------- | -------------------- | ----------- |
| `type`      | `string`             | `type` MUST be `string`. |
| `minLength` |                      | `minLength` MUST be present. |
| `maxLength` |                      | `maxLength` MUST be present. |
| `pattern`   |                      | `pattern` SHOULD be present unless character constraints cannot be expressed in a regular expression. |
{: caption="Schema guidance for strings in requests" caption-side="bottom"}


## Date
{: #date}

The date type specifies a particular year, month, and day.

### Response format
{: #date-return-format}

A date value MUST be returned as a JSON string containing a date in the format `YYYY-MM-DD`,
matching the `full-date` format as [specified by RFC
3339](https://datatracker.ietf.org/doc/html/rfc3339#section-5.6){: external}.

In this format:

*  `YYYY` is a four-digit year value
*  `MM` is a two-digit month value (`01`-`12`)
*  `DD` is a two-digit day-of-the-month value (`01`-`31`)

| Constraint  | Recommended value | Description |
| ----------- | ----------------- | ----------- |
| `type`      | `string`          | `type` MUST be `string`. |
| `format`    | `date`            | `format` MUST be `date`. |
{: caption="Schema guidance for dates in responses" caption-side="bottom"}

### Request formats
{: #date-accepted-formats}

A date value matching the returned format MUST be considered a valid date. All other values MUST NOT
be considered valid. An invalid date in the request body MUST be rejected with a `400` status code
and appropriate error response model.

An invalid date in a query parameter SHOULD be rejected with a `400` status code and appropriate
error response model.

| Constraint  | Recommended value | Description |
| ----------- | ----------------- | ----------- |
| `type`      | `string`          | `type` MUST be `string`. |
| `format`    | `date`            | `format` MUST be `date`. |
{: caption="Schema guidance for dates in requests" caption-side="bottom"}


## Date/Time
{: #datetime}

The date/time type specifies a particular date, time, and timezone.

### Response format
{: #date-time-return-format}

A date/time value MUST be returned as a JSON string containing a date/time value in the format
`YYYY-MM-DDTHH:mm:ssZ` or `YYYY-MM-DDTHH:mm:ss.sssZ`, matching the `date-time` format as [specified
by RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339#section-5.6){: external}.

In these formats:

*  `YYYY`, `MM`, and `DD` are as [described above](#date)
*  `T` is a literal, uppercase "T"
*  `HH` is the number of hours since midnight (`00`-`23`)
*  `mm` is the number of minutes since the start of the hour (`00`-`59`)
*  `ss` is the number of seconds since the start of the minute (`00`-`59`)
*  `sss` is the number of milliseconds since the start of the second (`000`-`999`)
*  `Z` is a literal, uppercase "Z", indicating no timezone offset (`±00:00` or UTC)

Date/time values SHOULD be returned with the same precision with which they are stored and MUST NOT
be returned with greater precision. That is, if a date/time value is stored internally with
second-precision, it MUST NOT be returned in the format `YYYY-MM-DDTHH:mm:ss.sssZ` where `sss` is
always `000`.

If a date/time value is stored with millisecond precision, but returned with second precision, the
value MUST be truncated to second precision before any client-facing operations, such as sorting.

Rounding a date/time value (instead of truncating the value) could change additional segments up to
and including the year. For example, `2020-12-31T23:59:59.999` rounds to `2021-01-01T00:00:00`.
Because higher-order segments could have special significance, reducing the precision of a
date/time value MUST be done with truncation and not rounding.
{: important}

| Constraint  | Recommended value                                         | Description |
| ----------- | --------------------------------------------------------- | ----------- |
| `type`      | `string`                                                  | `type` MUST be `string`. |
| `format`    | `date-time`                                               | `format` MUST be `date-time`. |
| `minLength` | `20` for second precision, `24` for millisecond precision | `minLength` MUST be present. |
| `maxLength` | `20` for second precision, `24` for millisecond precision | `maxLength` MUST be present. |
{: caption="Schema guidance for date/times in responses" caption-side="bottom"}

### Request formats
{: #date-time-accepted-formats}

A date/time value which case-insensitively matches either of the valid date/time response formats
MUST be considered a valid date/time value. A date/time value provided by the client SHOULD NOT be
required to have the same precision—either second or millisecond—as the field returns.

Additionally, a date/time value provided as input SHOULD be permitted to supply an explicit timezone
offset in place of a literal `Z`. This offset MUST be formatted as `+HH:mm` or `-HH:mm` where:

*  `HH` is the number of whole hours by which the timezone differs from UTC
*  `mm` is the number of additional minutes by which the timezone differs from UTC

In particularly delicate situations where input must be guaranteed to be correct, input with no
timezone offset (that is, input in UTC as indicated by a literal `Z`) MAY be required.

All other values MUST NOT be considered valid. An invalid date/time value in the request body MUST
be rejected with a `400` status code and appropriate error response model.

An invalid date/time value in a query parameter SHOULD be rejected with a `400` status code and
appropriate error response model.

| Constraint  | Recommended value | Description |
| ----------- | ----------------- | ----------- |
| `type`      | `string`          | `type` MUST be `string`. |
| `format`    | `date-time`       | `format` MUST be `date-time`. |
| `minLength` | `20`              | `minLength` MUST be present. |
| `maxLength` | `29`              | `maxLength` MUST be present. |
{: caption="Schema guidance for date/times in requests" caption-side="bottom"}


## CRN
{: #crn}

The CRN type provides for a Cloud Resource Name. CRN values MUST conform to the CRN specification
and MUST uniquely identify a resource among all resources.

It is not required that all models have CRN fields. However, a model which has an associated CRN
SHOULD include it in a field named `crn`. Any other field SHOULD NOT contain a CRN.

CRN fields MUST NOT be used in place of identifiers and MUST NOT be used to identify a resource in a
URI path segment.

### Response format
{: #crn-return-format}

For responses, a CRN field SHOULD have a documented maximum length of 512 characters. A CRN field
SHOULD have a permissible character set limited to printable ASCII and SHOULD have a documented
regular expression pattern which CRNs are guaranteed to match.

A CRN value MUST be returned as a JSON string and MUST be returned in consistent case. Individual
segments which might otherwise be case-insensitive (such as UUIDs) SHOULD be returned in lowercase.

| Constraint  | Recommended value | Description |
| ----------- | ----------------- | ----------- |
| `type`      | `string`          | `type` MUST be `string`. |
| `format`    | `crn`             | `format` MUST be `crn`. |
| `minLength` | `9`               | `minLength` SHOULD be present. |
| `maxLength` | `512`             | `maxLength` SHOULD be present. |
| `pattern`   | `^crn:v[0-9](:([A-Za-z0-9-._~!$&'()*+,;=@\/]|%[0-9A-Z]{2})*){8}$` | `pattern` SHOULD be present. |
{: caption="Schema guidance for CRNs in responses" caption-side="bottom"}

### Request formats
{: #crn-accepted-formats}

For requests, a CRN field MUST have a documented maximum length of 512 characters. A CRN field MUST
have a permissible character set limited to printable ASCII and SHOULD have a documented regular
expression pattern which CRNs are required to match.

Length, character set, and pattern constraints MUST be checked prior to any other processing of
CRNs included in requests. Failure to match constraints MUST cause the entire request to fail
with a `400` status code and appropriate error response model.

In request bodies and query parameters, CRN values SHOULD be matched case-sensitively.

| Constraint  | Recommended value | Description |
| ----------- | ----------------- | ----------- |
| `type`      | `string`          | `type` MUST be `string`. |
| `format`    | `crn`             | `format` MUST be `crn`. |
| `minLength` | `9`               | `minLength` MUST be present. |
| `maxLength` | `512`             | `maxLength` MUST be present. |
| `pattern`   | `^crn:v[0-9](:([A-Za-z0-9-._~!$&'()*+,;=@\/]|%[0-9A-Z]{2})*){8}$` | `pattern` MUST be present. |
{: caption="Schema guidance for CRNs in requests" caption-side="bottom"}


## Enumeration
{: #enumeration}

Enumerations are a category of types for specified sets of string values. Each individual
enumeration type is defined by the set of valid string values. For example, a model might define a
color enumeration allowing only the values `red`, `blue`, and `green`. Consequently, a field of the
color enumeration type could only contain one of those three values.

### Response format
{: #enumeration-return-format}

An enumeration value MUST be a member of the set of values defined for a particular enumeration
type. These values MUST be lower snake case strings and MUST begin with a letter and not a number.

In a response body, an enumeration field MUST be required. If the property is inapplicable to
certain states of the resource, this MUST be represented by an explicit value, such as
`inapplicable`.

| Constraint  | Recommended value    | Description |
| ----------- | -------------------- | ----------- |
| `type`      | `string`             | `type` MUST be `string`. |
| `enum`      |                      | `enum` MUST be an array of snake-case strings. |
{: caption="Schema guidance for enumerations in responses" caption-side="bottom"}

### Request formats
{: #enumeration-accepted-formats}

A case-insensitive match for one of the defined values for an enumeration MUST be considered a valid
value but MAY be rejected by other validation rules.

Character set constraints (only ASCII alphanumeric characters and the underscore character) MUST be
enforced prior to any case normalization[^case-normalization].

[^case-normalization]: Occasionally, case normalization may coerce a non-ASCII character into an
   ASCII character. This MUST NOT be allowed to occur.

All other values MUST NOT be considered valid. An invalid enumeration value in the request body MUST
be rejected with a `400` status code and appropriate error response model.

An invalid enumeration value in a query parameter SHOULD be rejected with a `400` status code and
appropriate error response model.

In a request body, an enumeration field MAY be optional. An optional enumeration field MUST have
either an explicit default value or an explanation of the strategy used to compute a value in the
property schema `description`.

| Constraint  | Recommended value    | Description |
| ----------- | -------------------- | ----------- |
| `type`      | `string`             | `type` MUST be `string`. |
| `enum`      |                      | `enum` MUST be an array of snake-case strings. |
{: caption="Schema guidance for enumerations in requests" caption-side="bottom"}


## Array
{: #array}

Arrays are a category of types for collections of same-type values. Each individual array type is
defined as a collection of values of a specific type. For example, a model might specify a field
with an [integer](#integer) array type. Consequently, this field could only contain an array with
zero or more [integer](#integer) values.

### Response format
{: #array-return-format}

For responses, an array field SHOULD have a documented minimum number and maximum number of items.
For the sake of backward-compatible future expansions, documented constraints MAY be more
permissive (allowing for fewer or more items) for responses than for requests.

In a response body, an array field MUST NOT be optional. A logically empty array MUST be
represented as a literal empty array (`[]`).

| Constraint  | Recommended value    | Description |
| ----------- | -------------------- | ----------- |
| `type`      | `array`              | `type` MUST be `array`. |
| `items`     |                      | `items` MUST be present. |
| `minItems`  |                      | `minItems` SHOULD be present. |
| `maxItems`  |                      | `maxItems` SHOULD be present. |
{: caption="Schema guidance for arrays in responses" caption-side="bottom"}

### Request formats
{: #array-accepted-formats}

For requests, an array field MUST have a documented minimum number and maximum number of items.

An array's item count constraints MUST be checked prior to any other processing of an array
included in a request. Failure to match constraints MUST cause the entire request to fail with a
`400` status code and appropriate error response model.

In a request body, an array field MAY be optional.

| Constraint  | Recommended value    | Description |
| ----------- | -------------------- | ----------- |
| `type`      | `array`              | `type` MUST be `array`. |
| `items`     |                      | `items` MUST be present. |
| `minItems`  |                      | `minItems` MUST be present. |
| `maxItems`  |                      | `maxItems` MUST be present. |
{: caption="Schema guidance for arrays in requests" caption-side="bottom"}


## Model
{: #model}

To learn about models, see the [Models](/docs/api-handbook?topic=api-handbook-models) section. Each
model—while itself a collection of fields with types—is also a type.

Because of this, one model can contain a field whose type is another model. For example, consider a
Server Model with a `processor` field. This field's type could be the Processor Model. Consequently,
an instance of the Server Model would contain an instance of the Processor Model in its `processor`
field.

[^json-keywords]: [In RFC
   7159](https://datatracker.ietf.org/doc/html/rfc7159#section-3){: external}, `true`, `false`, and
   `null` values are called "literal names," but in this document we refer to them as "JSON
   keywords."
