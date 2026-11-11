---
id: API-008
title: Choose an API authentication mechanism
type: decision-framework
strength: recommended
evidence: established
category: api-security
summary: Pick between session cookies, bearer tokens, API keys and mutual TLS based on who the caller is (browser, mobile app, partner server, internal service), how long access lasts and what a leaked credential can do.
lifecycle: [architecture, design]
standards: [OWASP-API-2023:API2]
tags: [api, authentication, bearer-token, api-key, mtls, session-cookie, decision]
contexts:
  - Designing a new API or adding a new class of client
  - Reviewing whether an existing mechanism fits its callers
relationships:
  - type: related_to
    target: AUTH-008
    note: That entry covers how users authenticate; this one covers how each API call carries identity.
  - type: related_to
    target: SESS-008
  - type: related_to
    target: AUTH-007
    note: Token-based options require full token validation.
  - type: related_to
    target: ARCH-005
  - type: related_to
    target: SESS-007
    note: Cookie-based authentication needs CSRF defenses.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
last_reviewed: "2026-11-07"
---

Choose by caller type and blast radius, not by habit. Whatever you choose, every call must still be authenticated and authorized server-side ([[API-001]]), over TLS ([[CRYPTO-004]]).

## Options

| Mechanism | Fits | Main strengths | Main risks |
|-----------|------|----------------|------------|
| Session cookie | First-party browser apps | Browser handles storage; server can revoke instantly | Needs CSRF protection ([[SESS-007]]) and cookie attributes ([[SESS-004]]) |
| Short-lived bearer token (for example from an authorization server) | Mobile, SPAs calling APIs, delegated access, third-party apps | Scoped, expiring, works across services | Theft allows replay; revocation harder; must validate fully ([[AUTH-007]]) |
| API key | Server-to-server integrations with low-sensitivity scope; identifying and metering a client | Simple | Usually long-lived, bearer-style, hard to scope; identifies an app, not a user |
| Mutual TLS or similarly bound credentials | Service-to-service, high-assurance partners | Credential tied to a private key; resists replay | Certificate lifecycle overhead; poor fit for browsers |

## Decision guidance

- Browser, same site as the API: session cookie.
- Acting on behalf of a user across origins or apps: short-lived access tokens with narrow scopes, plus refresh handling.
- Machine caller you control: mutual TLS or workload identity ([[ARCH-005]]), with automated rotation.
- Third-party developers: API keys to identify and rate-limit ([[API-002]]), combined with user-delegated tokens when user data is involved.
- Do not use an API key as the sole protection for sensitive user data.

## Always

Keep credentials out of URLs ([[DATA-006]]), give them an expiry or rotation path, scope them to least privilege ([[AUTHZ-002]]), and log their use without logging the secrets.
