---
id: API-003
title: Excessive data exposure in API responses
type: vulnerability
strength: strongly-recommended
evidence: established
category: api-security
summary: APIs that serialize whole internal objects and rely on the client to hide fields leak sensitive properties. Return only the fields each caller is authorized to see, chosen explicitly on the server.
lifecycle: [design, implementation, review, testing]
standards: [OWASP-API-2023:API3, CWE-200, OWASP-Top10-2021:A01]
tags: [api, data-exposure, serialization, response-filtering, dto]
contexts:
  - Endpoints that return database records or ORM objects directly
  - APIs serving several client types or roles from one model
  - Error and debug responses that include internal state
relationships:
  - type: depends_on
    target: API-005
    note: An explicit response schema is the main control.
  - type: related_to
    target: DATA-001
  - type: related_to
    target: DATA-005
  - type: related_to
    target: AUTHZ-003
    note: Property-level visibility is an authorization decision, like object-level access.
  - type: related_to
    target: API-004
    note: The write-side counterpart of the same unsafe object-to-wire coupling.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/200.html
last_reviewed: "2026-11-07"
---

## How it arises

A handler loads a record and returns it as-is, assuming the client will display only what it needs. The response then carries password hashes, internal IDs, other users' contact details, flags such as `is_admin`, or tokens. Anyone calling the API directly sees every field. New columns added later leak automatically.

## How to recognise it

- Handlers that return a model or database row without mapping it to a response shape.
- Generic "serialize everything" helpers, or exclusion lists (hide these fields) instead of inclusion lists.
- One response shape used for owners, other users and admins.
- Verbose errors or debug fields that expose internals ([[LOG-005]]).
- Filtering done in front-end code.

## How to prevent it

- Define an explicit response schema per endpoint and caller class, and build responses from it field by field ([[API-005]]). Prefer allow-lists so new fields are private until deliberately exposed.
- Decide property visibility by caller: owner, other user, staff. Apply the same authorization logic as for objects ([[AUTHZ-003]]).
- Apply [[DATA-001]]: do not fetch or return data the use case does not need, including in list and search endpoints.
- Mask or omit sensitive values (secrets, full identifiers) even for the owner unless needed.
- Add tests that assert the exact set of fields returned, so schema drift is caught.

```text
# Risky: whatever the model has goes out
return serialize(user)

# Better: explicit projection per audience
return { id: user.id, display_name: user.display_name }
```
