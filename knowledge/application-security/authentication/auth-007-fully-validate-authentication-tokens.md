---
id: AUTH-007
title: Fully validate authentication tokens (signature, issuer, audience, expiry)
type: pattern
strength: mandatory
evidence: established
category: authentication
summary: Before trusting a signed token, verify its signature with a pinned algorithm and expected key, then check issuer, audience, validity window and intended use. Never accept unsigned tokens or read claims before verification.
lifecycle: [design, implementation, review, testing]
standards: [CWE-347, CWE-287, OWASP-ASVS:V9, OWASP-Top10-2021:A07]
tags: [jwt, oidc, oauth, bearer-token, signature, audience, issuer, expiry]
contexts:
  - Services consuming JWTs, OIDC ID tokens or signed assertions
  - APIs that accept bearer access tokens
relationships:
  - type: mitigates
    target: AUTHZ-008
    note: Forged or confused tokens can carry elevated claims into authorization decisions.
  - type: supports
    target: API-001
  - type: related_to
    target: SESS-008
  - type: related_to
    target: ARCH-005
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/347.html
last_reviewed: "2026-11-07"
---

Treat every incoming token as attacker-controlled until a complete validation succeeds. Use a maintained library for this and configure it explicitly; do not hand-parse tokens or decode claims for decisions before the signature is verified.

**Validate all of these**

1. **Signature or MAC**, using a key you obtained from a trusted source (your configured issuer metadata or key set), never a key or algorithm named by the token itself. Allow-list the expected algorithm(s); reject `none`/unsigned tokens and prevent confusion between symmetric and asymmetric algorithms.
2. **Issuer**: exact match with the issuer you trust.
3. **Audience**: the token was minted for *your* service. Reject tokens issued for other services even if properly signed.
4. **Validity window**: expiry, and not-before where used, with small clock skew. Prefer short lifetimes.
5. **Token type and purpose**: an ID token is not an access token; a refresh token is not a bearer credential for APIs. Check scope or permissions for the operation ([[AUTHZ-005]]).
6. **Binding and replay**: for flows that use them, check the nonce, and consider sender-constrained tokens for high-value access.
7. **Key handling**: when fetching keys remotely, restrict to trusted URLs, cache sensibly, and handle rotation.

```
claims = verify(token,
        keys=trusted_keys_for(ISSUER), algs=["RS256"],
        issuer=ISSUER, audience=MY_SERVICE_ID,
        leeway=60s)               # verify() throws on any mismatch; fail closed
require claims.token_use == "access"
```

**Why this is mandatory:** a token whose signature, issuer, audience or validity window is not verified lets the caller assert any identity or reuse a token minted for another service. Each check is a cheap library call and no legitimate design relies on unverified claims, so there is no case for skipping them.

**Revocation:** stateless tokens stay valid until expiry. If you need immediate logout or revocation, use short lifetimes plus a revocation or session check ([[SESS-008]]).

Fail closed: any error in parsing or validation is a rejection ([[ARCH-004]]).

**Applies when** you accept self-contained or signed tokens, directly or via a gateway. If a gateway validates them, make sure downstream services cannot be reached around it ([[ARCH-005]]).
