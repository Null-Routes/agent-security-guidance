---
id: API-005
title: Enforce explicit request and response schemas
type: pattern
strength: strongly-recommended
evidence: established
category: api-security
summary: Define a schema for each operation's input and output and enforce it in code, rejecting unknown or malformed request data and emitting only declared response fields.
lifecycle: [design, implementation, testing]
standards: [CWE-20, OWASP-ASVS:V2, OWASP-ASVS:V4]
tags: [api, schema, validation, contract, openapi, allow-list]
contexts:
  - Any API that accepts structured bodies, parameters or headers
  - Services with multiple clients, or APIs consumed by third parties
  - Code generation or contract testing from an interface description
relationships:
  - type: mitigates
    target: API-004
    note: Rejecting undeclared fields stops unsafe binding.
  - type: mitigates
    target: API-003
    note: Declared response shapes prevent accidental field leakage.
  - type: implements
    target: INPUT-002
  - type: related_to
    target: API-007
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
last_reviewed: "2026-11-07"
---

Describe every operation's request and response in a machine-readable schema, and make the running service enforce it. The schema is the contract that decides what the server will accept and what it will reveal.

Without enforcement, handlers parse data ad hoc: types are assumed, extra fields slip through, sizes are unbounded and responses grow whatever the code happens to return. That produces injection inputs, [[API-004]] and [[API-003]].

## Request side

- Specify type, format, range, length, enum values and required/optional for every field, including path, query and header parameters.
- Reject unknown properties instead of ignoring them silently, and reject wrong content types.
- Bound strings, arrays, numbers and nesting depth ([[API-002]]).
- Validate before business logic and before any data reaches a query, command or template. Validation is the first layer, not a substitute for parameterization or encoding ([[INPUT-002]], [[INPUT-003]], [[INPUT-005]]).
- Fail with a generic client error that does not reveal internals ([[LOG-004]]).

## Response side

- Emit only declared fields, per audience, using the same schema mechanism.
- Include error responses in the schema so errors cannot leak stack traces or objects.
- In tests, assert that responses conform, so a new internal property cannot leak.

```text
operation updateProfile:
  request:  { display_name: string(1..80), email: email }   # additionalProperties: false
  response: { id: integer, display_name: string }
```

## Notes

Keep the schema as the single source for validation, documentation and tests so it does not drift. Schema checks establish shape and plausibility; they do not decide who may do what ([[API-001]]).
