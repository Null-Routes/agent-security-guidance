---
id: AUTHZ-007
title: Derive tenant context from verified identity and enforce tenant isolation
type: pattern
strength: strongly-recommended
evidence: established
category: authorization
summary: In multi-tenant systems, take the tenant from the authenticated identity, never from client input, and scope every data access, cache key, job and file path by that tenant at a layer handlers cannot skip.
lifecycle: [architecture, design, implementation, review, testing]
standards: [OWASP-API-2023:API1, OWASP-Top10-2021:A01, CWE-639, CWE-284]
tags: [multi-tenant, tenant-isolation, saas, data-isolation, tenant-id, authorization]
contexts:
  - SaaS or shared-infrastructure systems serving multiple customers
  - Adding endpoints, queries, caches or background jobs to a multi-tenant codebase
  - Reviewing code that accepts a tenant, organization or account identifier
relationships:
  - type: mitigates
    target: AUTHZ-004
    note: Cross-tenant access is the highest-impact form of broken object-level authorization.
  - type: depends_on
    target: AUTHZ-005
    note: Tenant scoping must live in a shared layer, not in each handler.
  - type: related_to
    target: AUTHZ-003
  - type: related_to
    target: API-004
    note: A tenant field must never be bindable from client input.
  - type: related_to
    target: ARCH-003
    note: A tenant boundary is a trust boundary.
sources:
  - https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
  - https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

Treat the tenant as part of the caller's verified identity, resolved on the server, and make it a mandatory scope of every data access. One missed filter exposes one customer's data to another, so isolation must not depend on each developer remembering it.

## Resolve the tenant from identity

- Establish the tenant from the authenticated session or validated token claim, or from a server-side membership lookup for the user ([[AUTH-007]]). Do not accept `tenant_id` from URL, body, query, header or a client-set claim as authority.
- If a user belongs to several tenants, let the client *select* one, then verify membership server-side on every request.
- Reject requests where a supplied tenant value disagrees with the verified one; do not silently prefer the client's.
- Exclude tenant fields from input binding ([[API-004]]).

## Scope every access

- **Queries:** every read, update and delete includes the tenant predicate, so an id from another tenant yields not-found.
- **Enforce below the handlers:** use a data-access layer, repository wrapper, or database row-level policy that injects the tenant and refuses unscoped queries ([[AUTHZ-005]]). Then add object-level checks inside the tenant ([[AUTHZ-003]]).
- **Everything else keyed by tenant:** cache keys, search indexes, object-storage paths, queues and message topics, background-job payloads, file names, rate limits, encryption keys where applicable, and exports and reports.
- **Joins and foreign keys:** a child row referencing a parent must be validated as the same tenant when created.
- **Admin and support access:** an explicit, audited cross-tenant path with its own rule ([[LOG-001]]).

```text
repo.find(model, id):
  return db.query(model, where: id = :id AND tenant_id = ctx.tenant_id)

ctx.tenant_id comes from verified identity, set once per request; unscoped query -> error
```

## Test it

Create two tenants with similar data. For every endpoint, call with tenant A's credential using tenant B's identifiers, parent ids, and filters; expect not-found or forbidden. Test lists, search, exports and background jobs too.

Stronger isolation (separate schemas, databases or deployments per tenant) reduces blast radius for sensitive customers but adds operational cost; the checks above still apply.
