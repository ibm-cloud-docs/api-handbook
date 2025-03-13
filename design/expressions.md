---

copyright:
  years: 2025
lastupdated: "2025-03-10"

subcollection: api-handbook

---

# Expressions
{: #expressions}

APIs that require clients to specify expressions MUST use the [Common Expression Language
(CEL)](https://github.com/google/cel-spec/blob/master/doc/langdef.md){: external}. Other expression
languages SHOULD NOT be supported except to adhere to a domain-specific industry precedent.

Exception: APIs MAY support the [extended
filtering](/docs/api-handbook?topic=api-handbook-filtering#extended-filtering) language for
collections.{: note}

## Validation
{: #cel-validation}

For robustness, client-specified expressions MUST be parsed and checked with one of the following CEL
implementations[^add-implementations]:

- [cel-go](https://github.com/google/cel-go)
- [cel-cpp](https://github.com/google/cel-cpp)
- [cel-java](https://github.com/google/cel-java)

Additionally, each client-specified expression MUST be statically checked verbatim (without any
canonicalization) for:
- Maximum length (at least 100 characters, and at most 1000 characters)
- Expected result type
- Time and space complexity (which must be assessed in the context in which the expression
  will be evaluated)

Client-specified expressions that pass these checks MUST be accepted and used verbatim. By
extension, for expressions that are being stored for later evaluation, stylistic choices made by the
client (including leading whitespace or embedded comments) MUST be preserved.

If the client-specified expression is not evaluated with one of the CEL implementations (e.g.,
because it's translated to an existing domain-native expression language prior to evaluation), the
native expression language MUST meet the [CEL language
requirements](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#syntax){: external} of
32 nested expressions and 32 repetitions of self-recursive or repetitive rules, unless other
constraints (such as the maximum expression length) preclude reaching those limits.

As a safeguard, any CEL expression that takes longer than 500 milliseconds to execute MUST be
aborted, and treated as if it evaluated to `false`. Aborted CEL expressions SHOULD be communicated
to the customer (for instance, through a customer-facing log entry or by transitioning the resource
to a degraded health state, with an accompanying health reason).

500 milliseconds is the maximum; evaluation MAY be aborted earlier.{:note}

## Minimum support
{: #cel-minimum-support}

To provide consistency and familiarity, CEL environments MUST support a minimum subset of the
language definition. The minimum support depends on the types of values that can be included in
the expression:

| Types     | Concept           | Operators / Functions    | Example                  |
| :-------  | :---------------: | :----------------------: | :----------------------: |
| any       | equality          | `==`, `!=`               | `x == y`                 |
| any       | precedence        | `()`                     | `x && (y \|\| z)`        |
| any       | conditional       | `?:`                     | `x > y ? x : y`          |
| boolean   | logic             |  `&&`, `\|\|`, `!`       | `x \|\| !y`              |
| list, map | indexing          | `[]`                     | `l[0] == l[size(l)-1]`   |
| list, map | membership        | `in`                     | `i in [1, 2, 3]`         |
| number[^numbers] | arithmetic |  `+`, `-`, `*`, `/`, `%` | `i + j % k`              |
| number    | negation          | `-`                      | `-i`                     |
| number, string, bytes, time   | comparison | `<`, `<=`, `>`, `>=` | `i >= j`        |
| string    | matching | `contains()`, `startsWith()`, `endsWith()` | `s.contains(t)` |
| string, bytes, list | concatenation | `+`                | `s + t`                  |
| string, bytes, list, map | counting | `size()`           | `size(s) < 100`          |
| time[^times] | arithmetic     |  `+`, `-`                | `t2 - t1`                |
{: caption="CEL minimum support" caption-side="bottom"}

In addition:
- If a given type can be included in an expression, support for type literals MUST be provided as
  per the CEL language definition. For example, if floating-point values are supported, then `8.0`,
  `8.`, `8`, and `8e0` MUST all be recognized as `8.0`.
- Standard CEL casting functions MUST be supported between all supported types. For example, if both
  numbers and strings are supported, it MUST be possible to cast from any numeric type to a string,
  and vice-versa.

## Additional functions
{: #cel-additional-functions}

The following additional functions are standardized and MUST be used when applicable:

| Function              | Type                           | Description                 |
| --------------------- | ------------------------------ | --------------------------- |
| `ipInRange(ip, cidr)` | (`string`, `string`) -> `bool` | True if `ip` is in `cidr`   |
{: caption="CEL additional functions" caption-side="bottom"}

CEL environments MAY define additional functions. To adhere to CEL naming conventions, these
functions MUST be named in lower camel case with any initialisms treated as words (e.g.,
`isAscii()`, not `isASCII()`). Additionally, these functions MUST NOT use dynamic return types.

## Variables
{: #cel-variables}

Any context-specific data MUST be provided in variables which MUST be named in lower snake case. The
following additional variables are standardized and MUST be used when applicable:

| Variable           | Type       | Description                    |
| ------------------ | ---------- | ------------------------------ |
| `destination.ip`   | string     | Destination IP address for a packet |
| `destination.port` | number     | Destination port  for a packet |
| `request.body`     | bytes      | HTTP request body              |
| `request.time`     | time       | HTTP request timestamp         |
| `response.body`    | bytes      | HTTP or DNS response body      |
| `response.time`    | time       | HTTP or DNS response timestamp |
| `source.ip`        | string     | Source IP address for a packet |
| `source.port`      | number     | Source port for a packet       |
{: caption="CEL variables" caption-side="bottom"}

When CEL is used in contexts for which there is an implied resource scope (such as collection
filtering), CEL variables MUST adhere to the [graph fragment
pattern](/docs/api-handbook?topic=api-handbook-schemas#graph-fragment-pattern). For example, a
collection expression that allows a resource to be filtered by its `type` property values MUST
provide a CEL variable named `type` for use in those filter expressions.

Exception: If the property name is a [CEL reserved
identifier](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#syntax){: external}, it
MUST be surrounded by double-underscores. For example, the property `namespace` (which is reserved
in CEL) MUST have the CEL variable name `__namespace__`.{:note}

APIs MAY support the [extended
filtering](/docs/api-handbook?topic=api-handbook-filtering#extended-filtering) language for
collections.{: note}

Functions and variables use separate namespaces in CEL. Therefore, a variable named `size` would not
collide with the CEL `size()` function.{: note}

## Versioning
{: #cel-versioning}

When CEL is used in contexts that imply a resource scope, an API version MUST accompany the CEL
expression, which MUST be used when the expression is evaluated. This ensures that as properties
within a resource schema evolve (such as by renaming, removing, or refactoring), the CEL variables
that refer to those properties remain interpreted according to the client's original intent. This
also ensures that CEL expressions do not need to be transformed by the server (e.g., to put it in
the "version" of the requesting client). As a result, if these CEL expressions need to be stored for
subsequent evaluation, the client's API version MUST be stored alongside the expression. Further,
before removing support for an older API version, a transition plan MUST be provided to enable
customers to refresh any persisted CEL expressions to a newer API version.

[^add-implementations]: Service teams may request approval for other CEL implementations, for
   example so CEL can be used by an additional language runtime.
[^numbers]: Numbers consist of the `int`, `uint`, and `double` CEL types.
[^times]: Times consist of the `timetamp` and `duration` CEL types.
