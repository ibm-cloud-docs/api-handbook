---

copyright:
  years: 2019
lastupdated: "2019-12-01"

subcollection: api-handbook

---

# Models
{: #models}

A model represents a [type](/docs/api-handbook/design/types.html) of resource and defines the
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

* Name
* Type
* Meaning: What the field represents
* Optionality: If the field is required to be present in every instance
* Nullability: If the field may contain an explicitly `null` value
* Mutability: If the field may be explicitly changed via the API
* Rules: Any constraints on the field's values in addition to ones imposed by its type

### Field types
{: #field-types}

Types constrain the kinds of values a field can contain. Each field must support only one of the
below types. Note that most field types are stricter than their base JSON type. Each type below is
linked to more information.

* An [Identifier](/docs/api-handbook/design/types.html#identifier) field contains a string value uniquely identifying a
  resource.
* A [Boolean](/docs/api-handbook/design/types.html#boolean) field contains a `true` or `false` value.
* An [Integer](/docs/api-handbook/design/types.html#integer) field contains a whole number.
* A [Float](/docs/api-handbook/design/types.html#float) field contains a number that could have a decimal point or scientific
  notation.
* A [String](/docs/api-handbook/design/types.html#string) field contains a free-form string.
* A [Date](/docs/api-handbook/design/types.html#date) field contains a specifically-formatted date string.
* A [Date/Time](/docs/api-handbook/design/types.html#datetime) field contains a specifically-formatted date and time string.
* A [CRN](/docs/api-handbook/design/types.html#crn) field contains a Cloud Resource
	Name.
* An [Enumeration](/docs/api-handbook/design/types.html#enumeration) field contains one of a predefined set of lower snake
  case strings.
* A [Model](/docs/api-handbook/design/types.html#model) field contains a nested object. This object must be an instance of a
  specific model; the field's definition must specify this model.
* An [Array](/docs/api-handbook/design/types.html#array) field contains an array which itself contains zero or more values.
  Each value must conform to a specific type (excluding the Array type itself); the field's
  definition must specify this type.
