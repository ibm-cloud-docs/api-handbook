---

copyright:
  years: 2021-2022
lastupdated: "2022-01-13"

subcollection: api-handbook

---

{:external: .external}

# Authorization
{: #authorization}

This section covers authorization, or access control, best practices for IBM Cloud APIs. Other
areas of interest include [authentication](/docs/api-handbook?topic=api-handbook-authentication) and
managing authorization using IBM Cloud [Identity and Access Management
(IAM)](https://cloud.ibm.com/iam/overview){: external}.

Authorization MUST be determined using platform-standard IAM tokens. Permission to perform a
specific action on a specific resource MUST be computed with official IAM libraries. Outside of
official IAM libraries, services SHOULD NOT compute permissions.

APIs should not allow anonymous access with the limited exception for publicly available resources
like documentation or samples. All other requests must require platform-standard IAM interaction.

APIs MUST document the access requirements for operations and keep this information up to date. This
documentation must include the IAM permissions needed for a request, as well as specific conditions
that require additional scoped permissions.

For example: 
> `POST /volumes` requires `is.snapshot.snapshot.operate` IAM action if a `source_snapshot` is
> specified in the request body.

Per the [resource-oriented
design](https://cloud.ibm.com/docs/api-handbook?topic=api-handbook-resources){: external} best practices, API
authors should consider implicit actions on non-primary resources that may be caused by the
relationships between resources. APIs should also communicate this via unambiguous [status
codes](/docs/api-handbook?topic=api-handbook-status-codes).

Users can [manage their
permissions](https://cloud.ibm.com/docs/account?topic=account-access-getstarted){: external} using IAM, which
regulates the scope and access allocated to each API key.

## Additional resources
{: #additional-resources}

*  [Authorization paradigms](/docs/api-handbook?topic=api-handbook-authorization)
*  [IBM Cloud Identity and Accesss Management](/docs/account?topic=account-iamoverview)
*  [SDK Authentication
   Guidelines](https://github.com/IBM/ibm-cloud-sdk-common#authentication){: external}
