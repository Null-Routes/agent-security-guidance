---
id: AUTHZ-001
title: Deny by default
type: principle
strength: strongly-recommended
evidence: established
category: authorization
summary: Access is refused unless a rule explicitly grants it. Resources, routes, roles and new features start with no access, so a missing or failed check results in denial instead of exposure.
lifecycle: [architecture, design, implementation, review]
standards: [OWASP-Top10-2021:A01, NIST-SP-800-53:AC-3, CWE-862, OWASP-ASVS:V8]
tags: [deny-by-default, access-control, allow-list, authorization, fail-closed]
contexts:
  - Designing the access model for a new service or feature
  - Adding endpoints, resources or roles to an existing system
  - Reviewing code where authorization is done ad hoc
relationships:
  - type: supports
    target: AUTHZ-005
    note: A central enforcement point is where the default-deny rule lives.
  - type: related_to
    target: AUTHZ-002
    note: Deny by default decides what happens without a rule; least privilege decides how broad each grant is.
  - type: related_to
    target: ARCH-004
    note: Errors in the access decision must resolve to denial.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
  - https://owasp.org/Top10/A01_2021-Broken_Access_Control/
  - https://cwe.mitre.org/data/definitions/862.html
last_reviewed: "2026-11-08"
---

Treat every request, resource and action as forbidden until an explicit rule allows it. Absence of a rule, an unknown role, a policy lookup error or a timeout must all produce a denial.

**Why:** the alternative is a list of things you remembered to block. Every forgotten route, new field or new role then becomes an exposure, and attackers look for exactly those gaps.

## What to do

- Build the access decision as an allow-list: `allow` only when a matching grant exists; otherwise `deny`. Do not write rules of the form "deny if on the blocklist".
- Make new routes, jobs, queues, files and admin tools inaccessible until a rule is attached. A route with no declared rule should fail at startup or in tests, not serve traffic.
- Give new accounts, roles and tokens no permissions by default; grant only what the task needs ([[AUTHZ-002]]).
- Wrap the decision so that exceptions, unreachable policy services and malformed input return deny. Document any deliberate fail-open case and who approved it ([[ARCH-004]]).
- Keep public access as an explicit, reviewable declaration (for example a named "anonymous" rule), not the absence of one.

```text
decide(subject, action, resource):
  try:
    grants = policy.lookup(subject, action, resource)
  except any_error:
    return DENY
  return ALLOW if grants.non_empty else DENY
```

## Applies when

Always for authorization decisions. It does not replace checking the specific object ([[AUTHZ-003]]); a rule that allows "any authenticated user to read documents" is still a grant that must be intended. Enforce the rule in one place where possible ([[AUTHZ-005]]).
