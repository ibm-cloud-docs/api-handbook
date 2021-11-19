---

copyright:
  years: 2020
lastupdated: "2020-03-05"

subcollection: api-handbook

---

# Long-running operations
{: #long-running-operations}

## Overview
{: #overview}

A long-running operation is an operation that returns a response with HTTP status code `202` to indicate that
the processing for the request will be [performed asynchronously (RFC 7231)][rfc-7231].

An operation that returns a `202` status code SHOULD NOT return any other 2xx status code.
In other words, an operation SHOULD NOT be _potentially_ asynchronous, using the status code to specify
that processing has already completed.

Here we do not consider operations that return a `200` to signify that a change to a "desired state"
has been accepted but will be achieved asynchronously.

Long-running operations MUST provide a means for the client application to determine when an operation has
completed and the outcome (success or failure) of the processing.
There are two general approaches to providing operation status -- an operation-based approach and
a resource-based approach.

In the operation-based approach, the response to the original request MUST contain an operation resource,
and the service MUST support a `GET` operation on the operation resource that returns the operation status.
The service MAY also support other operations on the operation resource, such as update or cancel.

The resource-based approach may be used in situations where the resource contains a status field
whose value is updated as a result of the operation and is thereafter fixed.
A typical scenario is the creation of a resource whose status field indicates when the creation process has completed.
When the resource-based approach is used, the response to the original request MUST contain a representation of the
subject resource including the resource status field.
The normal `GET` operation for the resource can then be used to poll for the completion of the operation.

## Operation-based long-running operations
{: #operation-based-long-running-operations}

The operation resource MUST contain a status field and this field SHOULD be named `status`.
The status field SHOULD be defined as an enumeration that includes the following values:
- `not_started` - the processing for the operation has not started
- `succeeded`   - the processing for the operation has completed successfully
- `failed`      - the processing for the operation failed.

The enumeration for the status field SHOULD also have a value that indicates "in progress" --
this could be `in_progress` or a more specific value for the operation, such as `creating`.

If the operation status indicates that the operation failed, the operation SHOULD include an `errors` array
with one or more [error models](/docs/api-handbook?topic=api-handbook-errors#error-model) that describe
the error(s) encountered in the operation processing.

The operation resource SHOULD contain an `href` property that matches the value of the `Location` header
in the response to the original request.

For operations that result in, or manipulate, a resource the service MUST include the target resource location
in the status upon operation completion.

Services MAY add additional, API specific, fields into the operation resource.

### Operation resources
{: #operation-resources}

Services MAY provide an `/operations` collection.

Services that provide the "/operations" resource MUST provide `GET` semantics.
`GET` MUST enumerate the set of operations, following standard pagination, sorting, and filtering semantics.

The service MUST NOT depend upon the client to consume the final status of the operation.
In other words, the service must handle "fire and forget" client behavior.
However, the service must preserve the operation result for a reasonable amount of time, for example 24 hours,
to allow the client time to consume it.

### Cancellation
{: #cancellation}

Services MAY support operation cancellation by exposing `DELETE` on the operation.
If supported, `DELETE` operations MUST be idempotent.
Operations that support cancellation MUST sufficiently describe their cancellation such that
the state of the system can be accurately determined, and any compensating actions may be run.

## Resource-based long-running operations
{: #resource-based-long-running-operations}

Any resource that could be the subject of a long-running operation MUST have a status field and this field SHOULD be named `status`.
The status field SHOULD be defined as an enumeration and all potentially long-running operations MUST
clearly describe how the completion of the operation is signaled with the value stored in the status field.

## Retry-After
{: #retry-after}

Long-running operations SHOULD return a `Retry-After` header in the original response that contains the number of seconds that
the client should wait before trying to get the status of the operation, to avoid excessive polling.
A `Retry-After` header MAY also be included in the response to `GET` requests on an operation resource.

[rfc-7231]: https://tools.ietf.org/html/rfc7231#section-6.3.3
