---
id: AUTH-008
title: Choose an authentication approach (local credentials, federated, passwordless)
type: decision-framework
strength: recommended
evidence: established
category: authentication
summary: Decide between locally managed passwords, federated sign-in through an identity provider, and passwordless or passkey authentication based on who the users are, assurance needs and your capacity to operate credential storage safely.
lifecycle: [architecture, design]
standards: [NIST-SP-800-63B, OWASP-ASVS:V6]
tags: [sso, oidc, saml, passkeys, passwordless, federation, identity-provider]
contexts:
  - Starting a new application or product
  - Replacing or consolidating login mechanisms
relationships:
  - type: related_to
    target: AUTH-001
    note: Only needed when you keep local passwords.
  - type: related_to
    target: AUTH-002
  - type: related_to
    target: AUTH-007
    note: Federation depends on correct token and assertion validation.
  - type: related_to
    target: API-008
sources:
  - https://pages.nist.gov/800-63-3/sp800-63b.html
  - https://pages.nist.gov/800-63-3/sp800-63c.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
last_reviewed: "2026-11-07"
---

Pick the authentication model deliberately; each one moves risk and operational burden to a different place.

| Situation | Recommendation |
|-----------|----------------|
| Employees, partners, or customers of an organization that already has an identity provider | **Federate** (OIDC or SAML SSO). Centralizes MFA, offboarding and policy. |
| Consumer product where you want low friction and fewer secrets to protect | Offer **passkeys/passwordless** as the primary path, with social/federated login as an option. |
| You must own the identity (no suitable external provider, offline or closed environments, regulatory constraint) | **Local credentials**, done fully: [[AUTH-001]], [[AUTH-002]], [[AUTH-003]], [[AUTH-005]]. |
| Mixed audiences | Combine: federation for organizations, passkeys or local accounts for individuals, behind one internal identity model. |

**Decision criteria**

- **Assurance needed:** high-risk access favours phishing-resistant authenticators and a central provider enforcing them.
- **Operational capacity:** local passwords mean you run hashing, breach screening, throttling, recovery and support. Delegating to a mature provider removes much of that, but adds a dependency and a trust relationship.
- **Account lifecycle:** federation lets an organization disable one account and cut access everywhere. Local accounts need your own provisioning and offboarding.
- **Provider trust and availability:** external providers become part of your trust boundary and a possible single point of failure. Plan fallback and account-linking rules so that an unverified email claim from one provider cannot take over an existing account.
- **User experience and recovery:** passwordless shifts the problem to device loss; design recovery explicitly ([[AUTH-005]]).

Whichever you choose, validate tokens or assertions completely ([[AUTH-007]]), keep authorization in your own application ([[AUTHZ-005]]), and prefer a maintained, well-reviewed library or service over a custom implementation ([[CRYPTO-001]]).

Avoid mixing methods in a way that lets the weakest one bypass the strongest (for example, a password path alongside enforced SSO).
