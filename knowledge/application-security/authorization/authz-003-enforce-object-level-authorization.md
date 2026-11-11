---
id: AUTHZ-003
title: Enforce object-level authorization on every access
type: pattern
strength: mandatory
evidence: established
category: authorization
summary: For every read, write, delete and list of a specific record, the server must check that the verified caller is permitted to access that record. Never rely on the identifier being hard to guess or on the client to filter.
lifecycle: [design, implementation, review, testing]
standards: [OWASP-API-2023:API1, OWASP-Top10-2021:A01, CWE-639, CWE-862, CWE-863, OWASP-ASVS:V8]
tags: [object-level, idor, bola, ownership, authorization, record-access]
contexts:
  - Any endpoint, job or function that takes an object identifier from the caller
  - List, search, export and bulk operations
  - Reading related or nested objects reached through another object
relationships:
  - type: mitigates
    target: AUTHZ-004
  - type: depends_on
    target: AUTHZ-005
    note: Put the check in a shared enforcement layer so no handler can forget it.
  - type: related_to
    target: AUTHZ-007
    note: In multi-tenant systems the tenant boundary is the outermost object-level check.
  - type: related_to
    target: API-001
  - type: related_to
    target: API-003
    note: Property visibility follows the same caller-versus-object logic.
sources:
  - https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
  - https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/639.html
last_reviewed: "2026-11-08"
---

Before returning or changing any specific record, check that the authenticated caller may perform this action on this object. Authentication and a role check do not answer that question: "is a user" and "can see invoices" do not mean "can see invoice 4711".

This is mandatory because the omission is among the most common and most exploited API flaws ([[AUTHZ-004]]), the check is cheap, and no legitimate design needs an exception.

## What to do

1. Take the caller's identity from the verified credential, never from a user ID in the URL, body, header or hidden field.
2. Load the object, or its ownership and relationship data, from the server's own store.
3. Evaluate the decision for the identity, the action and that object (owner, group, share, role in the object's tenant), through the shared enforcement point ([[AUTHZ-005]]).
4. Deny with a response that does not reveal whether the object exists to unauthorized callers (a uniform not-found is common).

Apply the check to every path that reaches the object:

- reads, updates, deletes and state transitions, on every method and API version;
- **lists, search and export**: filter the query by permission so unauthorized rows are never selected, rather than fetching everything and hoping the client hides it;
- nested and related objects (`/orders/7/items/9` must check that item 9 belongs to order 7 *and* order 7 is visible to the caller);
- bulk and batch endpoints, which must check each element;
- background jobs and webhooks acting on behalf of a user.

```text
get_invoice(caller, id):
  inv = store.find(id)                       # may be absent
  if inv is None or not policy.allows(caller, "read", inv):
      return NOT_FOUND
  return inv

list_invoices(caller):
  return store.query(where: policy.visible_to(caller))   # filter in the query
```

Unpredictable identifiers (UUIDs) reduce guessing but are not authorization; keep the check regardless. Test with two accounts: account A must get a denial for every identifier owned by account B.
