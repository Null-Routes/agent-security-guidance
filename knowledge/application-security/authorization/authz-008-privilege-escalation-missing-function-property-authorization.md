---
id: AUTHZ-008
title: Privilege escalation through missing function-level or property-level authorization
type: vulnerability
strength: strongly-recommended
evidence: established
category: authorization
summary: Callers gain unauthorized capabilities when privileged functions are reachable by ordinary users or when sensitive properties, such as role or status, can be read or written without a permission check on that field or operation.
lifecycle: [design, implementation, review, testing]
standards: [OWASP-API-2023:API3, OWASP-API-2023:API5, OWASP-Top10-2021:A01, CWE-269, CWE-862]
tags: [privilege-escalation, function-level, property-level, admin-endpoints, role, authorization]
contexts:
  - Admin, management and internal functions exposed on the same API as user functions
  - Update endpoints that accept partial objects
  - Systems where role, permission, price or status fields exist on user-editable models
relationships:
  - type: related_to
    target: AUTHZ-003
    note: Object-level checks answer which records; this entry covers which functions and fields.
  - type: related_to
    target: API-004
    note: Binding client fields onto models is a common route to escalating a role or flag.
  - type: related_to
    target: API-001
    note: Function-level checks are part of authorizing every request.
  - type: related_to
    target: AUTHZ-005
    note: Central policy makes privileged functions declarable and reviewable.
sources:
  - https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/
  - https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/
  - https://cwe.mitre.org/data/definitions/269.html
last_reviewed: "2026-11-08"
---

An authenticated, ordinary caller reaches more capability than intended. Two forms are common.

## Function-level

A privileged operation (create user, change role, refund, export all, reconfigure) is served to any authenticated caller because the check is missing, hidden only by the UI, or applied to some HTTP methods or paths but not others.

Typical causes: admin routes mounted alongside user routes with no distinct rule; checks on `GET` but not `PUT`/`DELETE`; an old API version or internal path left reachable; authorization inferred from URL shape ("/admin/") rather than enforced; a role check performed in the client.

## Property-level

The caller may access the object but not every field. Escalation happens when they can **write** privileged fields (`role`, `is_admin`, `owner`, `price`, `status`, `verified`) or **read** fields reserved for staff.

Typical causes: binding the whole request body onto a model ([[API-004]]); generic update endpoints; returning full records and trusting the client to hide fields ([[API-003]]).

## Prevention

- Classify functions by required privilege and declare the rule per route and method, with default deny ([[AUTHZ-001]], [[AUTHZ-005]]).
- Enforce on the server for every method, version and internal path; hiding menu items is not a control.
- Define separate input shapes per role and operation, listing only fields that caller may set. Set privileged fields from server logic, not input.
- Check permission per property for updates and for response fields; changes to roles or permissions should need an explicit privileged action and be audited ([[LOG-001]]).
- Do not trust role or privilege values from client-held data unless integrity-verified ([[AUTH-007]]).
- Test by calling each privileged function with a low-privilege account and by sending extra fields in updates.
