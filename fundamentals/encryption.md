---

copyright:
  years: 2021
lastupdated: "2021-06-30"

subcollection: api-handbook

---

# Encryption
{: #encryption}

Services MUST support HTTPS and MUST NOT allow clients to make unencrypted, HTTP-only requests.

Services MUST use TLS 1.2 or higher. Deprecated encryption protocols including (as of this writing)
TLS 1.0 and 1.1 and all versions of SSL MUST be fully disabled such that they are not supported for
clients that attempt to negotiate use of them.

Services MUST comply with all current IBM security policies on cipher support and certificate
management.
