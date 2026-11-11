---
id: SESS-008
title: "Choose a session model: server-side sessions or self-contained tokens"
type: decision-framework
strength: recommended
evidence: established
category: session-management
summary: Choose between server-side sessions with opaque identifiers and self-contained signed tokens by revocation needs, architecture, client types and the cost of a token staying valid until it expires.
lifecycle: [architecture, design]
standards: [OWASP-ASVS:V7]
tags: [sessions, jwt, stateless, revocation, opaque-token, refresh-token, decision]
contexts:
  - Designing authentication for a new web, mobile or API product
  - Considering a move from server-side sessions to stateless tokens, or the reverse
relationships:
  - type: related_to
    target: AUTH-007
    note: Self-contained tokens must be fully validated on every use.
  - type: related_to
    target: API-008
    note: That entry chooses the credential type per caller; this one chooses how state is held.
  - type: related_to
    target: SESS-003
    note: Invalidation is the main differentiator.
  - type: related_to
    target: SESS-001
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

Default to a server-side session with an opaque identifier unless a specific need justifies self-contained tokens. State where each choice keeps its risk before committing.

| | Server-side session (opaque ID) | Self-contained token (signed, for example JWT) |
|---|---|---|
| State | Held by server; identifier is a lookup key | Claims carried in the token; verified by signature |
| Revocation | Immediate: delete the record ([[SESS-003]]) | Valid until expiry unless you add a denylist, session lookup or "valid after" check |
| Scaling | Needs shared session storage | Verifiable by any service holding the key; no lookup |
| Changing permissions | Effective on next request | Stale until token is reissued |
| Typical fit | First-party browser apps, admin consoles, anything needing instant logout | Service-to-service, delegated access, federated scenarios, short-lived API calls |
| Main risks | Store availability and protection; fixation, CSRF ([[SESS-006]], [[SESS-007]]) | Validation mistakes ([[AUTH-007]]), key management ([[CRYPTO-006]]), long-lived leaked tokens, tokens in script-readable storage |

## Decision guidance

- **Need immediate logout, per-session control or role changes taking effect at once:** server-side sessions, or tokens with a session check.
- **Many independent services verifying identity without a central lookup:** self-contained tokens with **short lifetimes** (minutes), and refresh tokens that are revocable and rotated.
- **Browser clients:** prefer a protected cookie ([[SESS-004]]) holding an opaque identifier or a short-lived token; avoid keeping bearer tokens in script-readable storage.
- **Hybrid:** opaque session at the edge, short-lived internal tokens minted from it per request.

Do not choose stateless tokens only because they feel simpler: you then rebuild revocation, rotation and key handling. Do not put secrets in token payloads, because signed is not encrypted. Whichever you choose, set timeouts ([[SESS-005]]) and generate identifiers securely ([[SESS-001]]).
