---
copyright:
  years: 2021
lastupdated: "2021-11-17"
subcollection: api-handbook
---

{:note: .note}

# Operations
{: #operations}

## Operation IDs
{: #operation-ids}

Each operation in an OpenAPI document MUST have a unique `operationId` value. These values provide
stable, meaningful handles for operations. For example, `operationId` values can be used as anchors
in API documentation and function names in generated code.

Each `operationId` value SHOULD be a lower snake case string, with the convention of
`<verb>_<noun>` where `<verb>` relates to the nature of the operation, and `<noun>` matches a
resource type as it's represented in a path segment. The plurality of the noun MUST agree with the
number of resources operated on. For example, a `POST /reticulated_splines` operation to create a
reticulated spline would have an `operationId` of `create_reticulated_spline`.

Parent resource types in a path SHOULD be included as qualifiers in the noun used for the
`operationId`. For example, `DELETE /farms/{farm_id}/barns/{id}` would have an `operationId` of
`delete_farm_barn`.

The success status codes in the examples below are defined for synchronous operations only. Refer
to the section on [long-running
operations](/docs/api-handbook?topic=api-handbook-long-running-operations) for guidance on
operations that return `202 Accepted`.
{: note}

### Standard operations
{: #standard-operations}

The following conventions for `operationId` values SHOULD be used for standard operations on
resources where possible.

| Verb     | Example method and path | Success status code | Recommended operation ID | Description |
| -------- | ----------------------- | ------------------- | ------------------------ | ----------- |
| `list`   | `GET /albums`           | `200 OK`            | `list_albums`            | Retrieve a collection of _albums_ |
| `create` | `POST /albums`          | `201 Created`       | `create_album`           | Create a new _album_ |
| `get`    | `GET /albums/{id}`      | `200 OK`            | `get_album`              | Retrieve the _album_ identified by `{id}` |
| `update` | `PATCH /albums/{id}`    | `200 OK`            | `update_album`           | Update the _album_ identified by `{id}` |
| `delete` | `DELETE /albums/{id}`   | `204 No Content`    | `delete_album`           | Delete the _album_ identified by `{id}` |

### Resource replacement operations
{: #resource-replacement-operations}

The following conventions for operations to replace one or more resources SHOULD be used where
applicable.

| Verb      | Example method and path | Success status code | Recommended operation ID | Description |
| --------- | ----------------------- | ------------------- | ------------------------ | ----------- |
| `replace` | `PUT /symptoms/{id}`    | `200 OK`            | `replace_symptom`        | Replace the _symptom_ identified by `{id}` |
| `replace` | `PUT /symptoms`         | `200 OK`            | `replace_symptoms`       | Replace all _symptoms_ in a collection |

### Resource binding operations
{: #resource-binding-operations}

The following conventions for operations on bindings between resources SHOULD be used where
applicable.

The verbs `unset` and `remove` MUST NOT be used except for an operation that is trivially
reversible.

The noun in a resource binding operation SHOULD match the related resource type as it's represented
in the binding context. For example, even if `PUT /accounts/{id}/administrator` accepts a payload
identifying a user, the `operationId` would be `replace_account_administrator`.

Typically, the modeling of a binding intrinsically defines only a single direction of cardinality.
For example, if an account's binding to an administrator is modeled as
`/accounts/{id}/administrator`, it can be inferred that a given account has a single administrator.
However, depending on other factors, the user might be restricted to be the administrator for just
that account, or may be allowed to also be an administrator for other accounts.
{: note}

#### A resource's required binding to one other resource
{: #required-binding-to-one-other-resource}

The following convention SHOULD be used where a resource has a required binding to exactly
one other resource. In the example used, a club must have exactly one treasurer.

| Verb      | Example method and path     | Success status code | Recommended operation ID | Description |
| --------- | --------------------------- | ------------------- | ------------------------ | ----------- |
| `replace` | `PUT /clubs/{id}/treasurer` | `200 OK`            | `replace_club_treasurer` | Replace an existing binding with one between a _treasurer_ identified in the request body and the _club_ identified by `{id}` |

#### A resource's optional binding to one other resource
{: #optional-binding-to-one-other-resource}

The following convention SHOULD be used where a resource has an optional binding to (at most)
one other resource. In the example used, a hero might have one sidekick, but no more than one.

| Verb      | Example method and path        | Success status code                             | Recommended operation ID | Description |
| --------- | ------------------------------ | ----------------------------------------------- | ------------------------ | ----------- |
| `set`     | `PUT /heroes/{id}/sidekick`    | `201 Created` or `200 OK` (if a binding exists) | `set_hero_sidekick`      | Create (or replace an existing binding with) a binding between a _sidekick_ identified in the request body and the _hero_ identified by `{id}` |
| `unset`   | `DELETE /heroes/{id}/sidekick` | `204 No Content`                                | `unset_hero_sidekick`    | Remove the binding between the _sidekick_ and the _hero_ identified by `{id}` |

The verbs `set` and `unset` SHOULD NOT be used except in a symmetrical pair of operations.

#### A resource's bindings to multiple other resources
{: #bindings-to-multiple-other-resources}

The following convention SHOULD be used where a resource can have bindings to multiple other
resources. In the example used, a conference can have many speakers.

| Verb      | Example method and path                             | Success status code                               | Recommended operation ID    | Description |
| --------- | --------------------------------------------------- | ------------------------------------------------- | --------------------------- | ----------- |
| `add`     | `PUT /conferences/{conference_id}/speakers/{id}`    | `201 Created` or `200 OK` (if the binding exists) | `add_conference_speaker`    | Create (or ensure) a binding between the _conference_ identified by `{conference_id}` and the _speaker_ identified by `{id}` |
| `remove`  | `DELETE /conferences/{conference_id}/speakers/{id}` | `204 No Content`                                  | `remove_conference_speaker` | Remove the binding between the _conference_ identified by `{conference_id}` and the _speaker_ identified by `{id}` |

The verbs `add` and `remove` SHOULD NOT be used except in a symmetrical pair of operations.

### Minimally represented resource operations
{: #minimally-represented-resource-operations}

The following conventions for operations on resources (especially child resources) that can each
be wholly represented by a unique string SHOULD be used where applicable.

| Verb      | Example method and path             | Success status code                                             | Recommended operation ID | Description |
| --------- | ----------------------------------- | --------------------------------------------------------------- | ------------------------ | ----------- |
| `add`     | `PUT /books/{id}/genres/{genre}`    | `201 Created` or `204 No Content` (if the child already exists) | `add_book_genre`         | Create (or ensure the existence) of the _genre_ described by `{genre}` for the _book_ identified by `{id}` |
| `remove`  | `DELETE /books/{id}/genres/{genre}` | `204 No Content`                                                | `remove_book_genre`      | Remove the _genre_ described by `{genre}` from the _book_ identified by `{id}` |
| `check`   | `GET /books/{id}/genres/{genre}`    | `204 No Content`                                                | `check_book_genre`       | Check for the existence of the _genre_ described by `{genre}` on the _book_ identified by `{id}` |

### Custom operations
{: #custom-operations}

Some operations do not fit one of the defined conventions. Typically these operations are either:

- Inherently functional — for example, an operation that converts text in a request payload to
  audio in a response
- Result in an unchanged end state — for example, an operation that reboots a virtual server 
  instance

A custom operation SHOULD use a `POST` or a `GET` method, following [the requirements for those
methods](/docs/api-handbook?topic=api-handbook-methods).
