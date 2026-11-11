---
id: AUTHZ-004
title: Broken object-level authorization (IDOR / BOLA)
type: vulnerability
strength: strongly-recommended
evidence: established
category: authorization
summary: When a server uses a caller-supplied object identifier without checking that the caller may access that object, changing the identifier exposes or alters other users' data. Fix it with a per-object check.
lifecycle: [design, implementation, review, testing, maintenance]
standards: [OWASP-API-2023:API1, OWASP-Top10-2021:A01, CWE-639, CWE-862, CWE-863]
tags: [idor, bola, access-control, identifiers, enumeration, vulnerability]
contexts:
  - APIs and pages whose URLs or bodies carry record identifiers
  - Mobile or single-page clients that call APIs directly
  - Reviewing endpoints that fetch, update or delete by identifier
relationships:
  - type: related_to
    target: AUTHZ-003
    note: This pattern is the fix; this entry describes the weakness.
  - type: related_to
    target: AUTHZ-007
    note: Cross-tenant data access is the same flaw at the tenant boundary.
  - type: related_to
    target: LOG-007
    note: Sequential access to many identifiers is a detection signal.
  - type: related_to
    target: API-001
sources:
  - https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
  - https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/639.html
last_reviewed: "2026-11-08"
---

Broken object-level authorization (called IDOR in web contexts and BOLA in API contexts) occurs when the server accepts an identifier from the client and acts on that object without verifying the caller's right to it. The attacker authenticates as themselves, then substitutes another identifier.

## How it arises

- The handler checks only that the caller is logged in, or has a role, then loads `id` from the request.
- The user ID is taken from the request (`/users/{id}/profile`, `{"owner": 42}`) rather than from the session or token.
- Reads are protected but updates, deletes, exports, file downloads or nested sub-resources are not.
- List and search endpoints return everything and rely on the client to filter.
- Authorization is added per handler by hand, so one is missed ([[AUTHZ-005]]).
- Sequential or guessable identifiers make enumeration trivial; unguessable ones only slow it.

## Recognising it

Review every route that takes an identifier and ask: where is the check that this caller owns or may access this record? Test with two accounts, replaying account A's requests with account B's identifiers, across all methods and also in bulk, GraphQL or batch forms. Unexpected success, or a different error for "exists but forbidden" than for "does not exist", signals a problem.

## Preventing it

- Enforce a per-object check on every access path, including list results ([[AUTHZ-003]]).
- Derive identity and tenant from the verified credential ([[AUTHZ-007]]).
- Centralize enforcement so new endpoints inherit it ([[AUTHZ-005]]).
- Add automated cross-account tests for each resource type.
- Use non-sequential identifiers as an extra layer only, and alert on enumeration patterns ([[LOG-007]]).
