---

copyright:
  years: 2021
lastupdated: "2021-06-30"

subcollection: api-handbook

---

{:external: .external}

# Security
{: #security}

## Security by design
{: #security-by-design}

Security considerations are a critical part of the design of any API. The following principles
SHOULD be applied during an service API's design and development:

*  Operations SHOULD be simple and narrowly scoped such that they that are easy to reason about,
   easy to test, and closely match specific authorizations.
*  Convenience features SHOULD be carefully weighed against the additional complexity they require,
   and eschewed where a strong justification is not present.
*  Features and data that are not solving a well-understood use case SHOULD NOT be included in a
   service API. 
*  Sensitive information SHOULD NOT be returned in more operations and resource contexts than
   necessary.
*  Where feasible, resources SHOULD be designed to be simple, easy to delete and recreate, and with
   immutable qualities. Large, complex, long-lived resources with large numbers of mutable
   properties SHOULD be avoided.
  
These approaches contain the attack surface of an API, and allow for more comprehensive security
testing.

## Attack vector analysis
{: #attack-vector-analysis}

When designing and developing a service API, industry resources on common attacks and attack
mitigation SHOULD be considered. Recommended resources include:

*  [OWASP API Security Top 10](https://owasp.org/www-project-api-security/){: external}
*  [NIST Guide to Secure Web
   Services](https://csrc.nist.gov/pubs/sp/800/95/final){: external}
*  MITRE attack matrices for
   [SaaS](https://attack.mitre.org/matrices/enterprise/cloud/saas/){: external}
   and [IaaS](https://attack.mitre.org/matrices/enterprise/cloud/iaas/){: external}

## See also
{: #see-also}

Important best practices with security implications are addressed across this handbook:

*  [Authentication](/docs/api-handbook?topic=api-handbook-authentication)
*  [Authorization](/docs/api-handbook?topic=api-handbook-authorization)
*  [Encryption](/docs/api-handbook?topic=api-handbook-encryption)
*  [Robustness](/docs/api-handbook?topic=api-handbook-robustness)
*  Validation for [formats](/docs/api-handbook?topic=api-handbook-format),
   [parameters](/docs/api-handbook?topic=api-handbook-uris#query-parameters),
   and [types](/docs/api-handbook?topic=api-handbook-types)
