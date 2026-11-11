---
id: API-004
title: Mass assignment from unsafe binding of client-supplied fields
type: vulnerability
strength: strongly-recommended
evidence: established
category: api-security
summary: Automatically binding request fields onto internal objects lets clients set properties they should never control, such as roles, ownership or prices. Bind only an explicit allow-list of writable fields.
lifecycle: [design, implementation, review, testing]
standards: [CWE-915, OWASP-API-2023:API3]
tags: [api, mass-assignment, binding, object-properties, privilege-escalation]
contexts:
  - Create and update endpoints that map JSON or form bodies onto models
  - Code that merges request data into stored objects in a loop or with a generic helper
relationships:
  - type: depends_on
    target: API-005
    note: Request schemas that reject unknown fields are the main preventive control.
  - type: related_to
    target: AUTHZ-008
    note: Setting a role or admin flag through binding is privilege escalation.
  - type: related_to
    target: INPUT-002
  - type: related_to
    target: API-003
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/915.html
last_reviewed: "2026-11-07"
---

## How it arises

To save effort, a handler copies every field of the request body onto a model, or hands the whole body to an update helper. The model has properties the client was never meant to set: `role`, `is_admin`, `owner_id`, `account_balance`, `verified`, `tenant_id`. An attacker adds those names to a normal request and the server accepts them.

## How to recognise it

- A body, form or query map passed directly to a constructor, `update` or `merge` function.
- Models that mix user-editable and server-controlled properties.
- Update endpoints where the client resends the full object returned by a GET.
- Blocklists of "dangerous" fields, which miss newly added properties.

```text
# Vulnerable: any field the client sends is written
user = load(id)
user.assign(request.body)
save(user)
```

## How to prevent it

- Define a dedicated input shape per operation that lists only writable fields, and copy those fields explicitly. Everything else is ignored or, better, rejected ([[API-005]]).
- Keep server-controlled values (ownership, roles, status, tenant, timestamps) out of input types and set them from the verified session or server logic ([[AUTHZ-007]] for tenant IDs).
- Use allow-lists, not blocklists.
- Check authorization for each property being changed; some fields are editable only by certain roles ([[AUTHZ-008]]).
- Test by sending extra fields in create and update calls and asserting they have no effect.

```text
# Safer: explicit writable fields
user.display_name = request.body.display_name
user.email        = validate_email(request.body.email)
save(user)
```
