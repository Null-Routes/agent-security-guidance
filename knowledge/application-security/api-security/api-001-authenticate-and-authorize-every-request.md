---
id: API-001
title: Authenticate and authorize every API request
type: pattern
strength: strongly-recommended
evidence: established
category: api-security
summary: Each API request must be authenticated (unless deliberately public) and authorized for the specific function and object on the server, on every call, not just at login or at a gateway edge.
lifecycle: [design, implementation, review]
standards: [OWASP-API-2023:API1, OWASP-API-2023:API2, OWASP-API-2023:API5, OWASP-Top10-2021:A01, CWE-306, CWE-862]
tags: [api, authentication, authorization, access-control, endpoints]
contexts:
  - Designing or adding any HTTP, RPC or GraphQL endpoint
  - Reviewing routes for missing access checks
  - Exposing internal APIs that browsers or third parties can reach
relationships:
  - type: implements
    target: AUTHZ-001
    note: Endpoints with no explicit access rule are rejected rather than served.
  - type: depends_on
    target: AUTHZ-005
    note: Enforcement should call one central decision point instead of ad hoc checks.
  - type: related_to
    target: AUTHZ-003
  - type: related_to
    target: AUTHZ-008
  - type: related_to
    target: AUTH-007
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
last_reviewed: "2026-11-07"
---

Make every API request pass through two server-side steps before any work happens: establish who the caller is, then decide whether that caller may perform this action on this resource. Do not rely on the client, the UI, network location, or a one-time check at login.

APIs expose each function and object directly, so one endpoint that skips a check is a complete bypass. Attackers enumerate routes, methods and identifiers without ever using the official client.

## What to do

- Treat endpoints as closed by default: a route is reachable only if it declares an authentication requirement and an authorization rule. Public endpoints should be an explicit, reviewed exception.
- Authenticate on each request using a credential the server can verify (see [[AUTH-007]] for token validation and [[API-008]] for mechanism choice).
- Authorize at two levels: the function (may this role call this operation?) and the object (may this caller touch *this* record?). See [[AUTHZ-003]] and [[AUTHZ-008]].
- Cover every method and version of a route, plus internal, admin, health, debug and batch endpoints. A forgotten `DELETE` or an old version is the usual gap.
- Take identity from the verified credential, never from a user ID in the body or URL.
- Route decisions through shared middleware or a policy service ([[AUTHZ-005]]) so new endpoints inherit enforcement.

```text
handler(request):
  caller = authenticate(request.credential)      # reject if missing/invalid
  if not policy.allows(caller, request.action, request.resource):
      return 403
  ...do the work...
```

## Applies when

Any API that is not intentionally anonymous. A gateway check is useful but does not replace the service's own object-level check, since the service knows who owns the data.
