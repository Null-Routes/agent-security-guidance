---
id: AUTHZ-005
title: Centralize authorization decisions in one enforcement point
type: pattern
strength: strongly-recommended
evidence: established
category: authorization
summary: Route every access decision through one shared layer, such as middleware, a policy module or a policy service, instead of scattering ad hoc checks across handlers, so rules are consistent, reviewable and inherited by new code.
lifecycle: [architecture, design, implementation, review]
standards: [OWASP-Top10-2021:A01, CWE-862, NIST-SP-800-53:AC-3, OWASP-ASVS:V8]
tags: [policy, enforcement-point, middleware, policy-engine, access-control, consistency]
contexts:
  - Designing the authorization layer of a new application
  - Refactoring a codebase with inline role checks
  - Adding endpoints or services that must follow existing rules
relationships:
  - type: supports
    target: AUTHZ-001
    note: One enforcement point is where default deny is implemented once.
  - type: related_to
    target: AUTHZ-003
    note: Object-level checks are executed through this layer.
  - type: related_to
    target: AUTHZ-006
    note: The access model determines what the central policy expresses.
  - type: related_to
    target: AUTH-008
    note: Authorization stays in your application even when identity is delegated.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
  - https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  - https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
last_reviewed: "2026-11-08"
---

Implement authorization as one reusable mechanism that every request path calls, with the rules held in one place. Handlers ask a question ("may this caller do this to that?") and obey the answer; they do not contain their own role logic.

**Why:** scattered `if user.role == ...` checks drift apart. One handler checks, another forgets, a third has a slightly different rule, and nobody can review the whole policy. Central enforcement makes absence of a check detectable and new endpoints safe by default.

## What to do

- Separate the **decision** (policy: who may do what) from the **enforcement** (where the answer is applied). Enforce at the service that owns the data; a gateway check may supplement it but does not replace it.
- Make the enforcement point unavoidable: middleware, interceptors, or a data-access layer that requires a decision. Routes with no declared policy should fail closed ([[AUTHZ-001]]).
- Pass the verified identity, action and resource to the policy; do not let the caller-supplied request supply the answer.
- Express policy as data or declarative code that can be reviewed, versioned and tested independently of handlers.
- Log denials and decisions in a consistent format ([[LOG-001]]).
- Apply the same layer to internal paths such as queues, scheduled jobs and service-to-service calls ([[ARCH-005]]).
- Test the policy with a table of subject, action, resource and expected result, including negative cases.

```text
middleware(request):
  caller = request.verified_identity
  decision = policy.decide(caller, request.action, request.resource_ref)
  if decision != ALLOW: return DENY
  next(request)
```

## Trade-offs

A remote policy service adds latency and an availability dependency; decide fail-closed behaviour and caching rules explicitly ([[ARCH-004]]). Resource-specific checks may still need data only the handler has, so provide a policy call that handlers invoke, not a free-form alternative.
