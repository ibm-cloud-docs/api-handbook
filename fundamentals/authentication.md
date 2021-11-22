---

copyright:
  years: 2021
lastupdated: "2021-06-30"

subcollection: api-handbook

---

# Authentication
{: #authentication}

This section covers authentication best practices for IBM Cloud APIs. Other areas of interest
include [authorization](/docs/api-handbook?topic=api-handbook-authorization) and IBM Cloud's
[Identity and Access Management (IAM)](/docs/account?topic=account-iamoverview) service.

IBM Cloud APIs MUST comply with [OAuth 2.0](https://oauth.net/2/) authentication standards and
accept Bearer tokens provided by IBM Cloud's [IAM](https://cloud.ibm.com/iam/overview) service.
Service APIs MUST accept authentication information exclusively in the HTTP `Authorization` header.

## Best practices
{: #best-practices}

IBM Cloud APIs MUST accept only platform-standard expiring IAM tokens. These tokens MUST be
validated using official IAM libraries. Outside of official IAM libraries, services SHOULD treat
tokens as opaque.

APIs SHOULD require authentication, though APIs providing publicly available information such as
templates or documentation MAY be made accessible without authentication.  The increased risk of
denial-of-service attacks MUST be considered and appropriately mitigated for any API accessible
without credentials.

Non-expiring credentials such as passwords or raw API keys MUST NOT be accepted in production
environments.

## Additional resources
{: #additional-resources}

* [API Authorization](/docs/api-handbook?topic=api-handbook-authorization)
* [IBM Cloud Identity and Accesss Management](/docs/account?topic=account-iamoverview)
* [SDK Authentication Guidelines](https://github.com/IBM/ibm-cloud-sdk-common#authentication)
