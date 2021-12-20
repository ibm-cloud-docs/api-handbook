---

copyright:
  years: 2019
lastupdated: "2019-12-01"

subcollection: api-handbook

---

{:note: .note}

# Models
{: #models}

A model represents a [type](/docs/api-handbook?topic=api-handbook-types) of resource and defines the
structure and behavior of objects that represent resources of that type. Such objects are called
_instances_ of the model. Every object returned or accepted by a service MUST be an instance of a
model.

In designing an API, the system must be decomposed into resources and each resource carefully
modeled. This process must be guided by how the consumer is expected to understand the system. It
should not be dictated by how the system itself represents or stores objects internally. The mapping
between models and internal structures — such as classes — must be well-behaved but it need not be
one-to-one.

An API's models should be described and implemented consistently in the API's specification and
documentation and in the service that provides the API. Where possible, automated tooling and
testing should be used to ensure this consistency.

## Fields
{: #fields}

A model specifies the following about each field:

*  Name
*  Type
*  Meaning: What the field represents
*  Optionality: If the field is required to be present in every instance
*  Nullability: If the field may contain an explicitly `null` value
*  Mutability: If the field may be explicitly changed via the API
*  Rules: Any constraints on the field's values in addition to ones imposed by its type

### Field types
{: #field-types}

Types constrain the kinds of values a field can contain. Each field must support only one of the
below types. Note that most field types are stricter than their base JSON type. Each type below is
linked to more information.

*  An [Identifier](/docs/api-handbook?topic=api-handbook-types#identifier) field contains a string
   value uniquely identifying a resource.
*  A [Boolean](/docs/api-handbook?topic=api-handbook-types#boolean) field contains a `true` or
   `false` value.
*  An [Integer](/docs/api-handbook?topic=api-handbook-types#integer) field contains a whole number.
*  A [Float](/docs/api-handbook?topic=api-handbook-types#float) field contains a number that could
   have a decimal point or scientific notation.
*  A [String](/docs/api-handbook?topic=api-handbook-types#string) field contains a free-form string.
*  A [Date](/docs/api-handbook?topic=api-handbook-types#date) field contains a
   specifically-formatted date string.
*  A [Date/Time](/docs/api-handbook?topic=api-handbook-types#datetime) field contains a
   specifically-formatted date and time string.
*  A [CRN](/docs/api-handbook?topic=api-handbook-types#crn) field contains a Cloud Resource Name.
*  An [Enumeration](/docs/api-handbook?topic=api-handbook-types#enumeration) field contains one of a
   predefined set of lower snake case strings.
*  A [Model](/docs/api-handbook?topic=api-handbook-types#model) field contains a nested object. This
   object must be an instance of a specific model; the field's definition must specify this model.
*  An [Array](/docs/api-handbook?topic=api-handbook-types#array) field contains an array which
   itself contains zero or more values. Each value must conform to a specific type (excluding the
   Array type itself); the field's definition must specify this type.

### Required, optional, and nullable fields
{: #required-optional-and-nullable}

#### In requests
{: #required-optional-and-nullable-in-requests}

If a field is optional in a request body, the behavior of omitting the field from a request MUST be
defined as either:

*  The use of a static value, defined in the property schema `default`
*  The use of a value otherwise computed by the system, with a strategy explained in the property
   schema `description`
*  Omission or removal[^omit-to-remove] from the subsequently observable resource state, with an
   explanation for why (and under what circumstances) a value is not required for the resource, in
   the property schema `description`
   
[^omit-to-remove]: Removal of an existing value applies only the case of a `PUT`-based resource
   replacement operation.

A `null` value for a field MUST NOT be accepted in a request body except in a [JSON merge patch
request](/docs/api-handbook?topic=api-handbook-methods#patch). A `null` value in a JSON merge
patch request body MUST NOT be accepted unless the field can be removed from the observable
resource state.

An operation MUST NOT accept any other sentinel value in a request body to omit or remove a field
from the subsequently observable state.

Omitted and `null` values offer similarly "empty" meaning, but the JSON merge patch type [employs
the `null` value as a sentinel](https://datatracker.ietf.org/doc/html/rfc7386#section-1) to remove
an existing value. The recommendation of JSON merge patch by this handbook provides precedent for a
client sending an explicit `null` value for a field to expect it to be subsequently omitted from
the observable resource state. It is therefore considered dangerous to accept a `null` value from a
client for a field that is required in the resource state reflected in responses. Finally, even for
a field in a `POST` or `PUT` request that is optional and omitted from the subsequently observable
state by default, it is both unnecessary and in violation of the principle of avoiding [input
canonicalization](/docs/api-handbook?topic=api-handbook-robustness#input-canonicalization) to
accept a `null` value.
{:note: .note}

#### In responses
{: #required-optional-and-nullable-in-responses}

A field in a response body SHOULD be required except where an aspect of a request or the state of a
resource might make a value inapplicable or unknown and no industry convention for a sentinel value
already exists for the value type.

If a field is optional in a response body, the circumstances where the field will be omitted and
the significance of the field's omission (or, conversely, the circumstances and significance of its
inclusion) MUST be well defined.

A field MUST NOT be omitted from a response body because an internal failure is currently preventing
the system from retrieving its value. An internal failure of the service to populate a value MUST
cause the entire request to fail with a `500` status code and appropriate error response model.

A field SHOULD NOT be omitted from a response body because the system hasn't yet computed its value.
If a field is optional for that reason, its omission MUST be limited to a specific and observable
resource state, such as a `status` value of `pending`.

If the omission of a field in a response body has an intuitive semantic, that semantic SHOULD be
honored. For example, if a response body field defines the date and time at which an event
occurred, and the field is optional, the omission of the field SHOULD indicate that the event has
never occurred.

The omission of an optional field from a response body SHOULD be determined by an aspect of the
request or predictable from and consistent with other aspects of observable state. For example, if a
property is inapplicable for a certain resource subtype, that subtype SHOULD be represented in a
clear way (generally a specific value of an enumeration field) that allows a client to infer when
the optional field will be omitted.

A field MUST NOT contain a `null` value in a response body. Rather, the absence of a value for a
field in the observable resource state SHOULD be represented by the omission of the field from the
response.

Permitting both omitted fields and `null` values in response bodies would present a usability
problem because of the potential for conflation by both human developers and JSON deserializers.
The choice of omission instead of sending explicit `null` values can be considered stylistic, but
is also supported by the use of JSON merge patch, [whose specification
notes](https://datatracker.ietf.org/doc/html/rfc7386#section-1) that its own use of `null` as a
sentinel limits its suitability to "describing modifications to JSON documents that primarily use
objects for their structure and do not make use of explicit `null` values."
{:note: .note}

### Examples
{: #property-examples}

Every primitive property in a schema MUST have a valid realistic `example` value. Example values
SHOULD be consistent with values that could appear in the system under normal usage conditions and
SHOULD NOT contain any obvious redactions or contrived abbreviations. Sensitive values (such as real
identifiers or employee usernames) from test accounts used to generate examples MUST be replaced
with dummy values that appear no less real.

For example, an example encryption key SHOULD be a real key, albeit one generated for no other
purpose than to be an example.

The example for each property SHOULD be compatible with `example` values for other properties in
the schema such that a schema example constructed from individual property examples is itself
valid. However, if two properties are themselves mutually exclusive in a schema, each of the
properties MUST still have an `example` value.

#### Object and array examples
{: #object-and-array-examples}

An example for an object or array MUST be embedded as a native JSON or YAML structure in an
OpenAPI definition and MUST NOT be represented as JSON wrapped in a string.
