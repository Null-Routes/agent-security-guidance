---
id: LOG-004
title: Return generic errors to clients and keep details server-side
type: pattern
strength: strongly-recommended
evidence: established
category: logging-monitoring
summary: Show clients a short, uniform error with a correlation ID, and record the full diagnostic detail in server-side logs. Install a global error handler so unhandled failures cannot leak internals.
lifecycle: [design, implementation, review]
standards: [CWE-209, OWASP-Top10-2021:A05]
tags: [error-handling, information-disclosure, correlation-id, global-handler, exceptions]
contexts:
  - building HTTP or RPC error responses
  - configuring production versus development error output
  - reviewing exception handling at service boundaries
relationships:
  - type: mitigates
    target: LOG-005
    note: Generic client errors remove the stack traces and internals that this weakness exposes.
  - type: implements
    target: ARCH-004
    note: Errors fail closed without revealing system state.
  - type: related_to
    target: AUTH-006
    note: Uniform responses also prevent account enumeration.
  - type: related_to
    target: LOG-001
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/209.html
last_reviewed: "2026-11-07"
---

Respond to failures with a minimal, consistent message and a correlation ID. Write the exception, stack trace, failing input and internal state to the server-side log under the same ID. Attackers use verbose errors to learn technology, schema, paths and validation rules ([[LOG-005]]).

**Do**
- Register a catch-all error handler at every entry point (HTTP, queue consumer, RPC) so no exception escapes with default framework output.
- Return an appropriate status code, a stable machine-readable error code, a human-friendly message, and the correlation ID.
- Log the full detail server-side with that ID, subject to [[LOG-002]] (do not log secrets).
- Use identical responses for cases that would otherwise reveal state, such as "unknown user" versus "wrong password" ([[AUTH-006]]) or "not found" versus "forbidden" where existence is itself sensitive.
- Turn off debug pages and detailed error modes in production, and verify that in deployment checks.
- Make sure error paths deny access rather than allow it ([[ARCH-004]]).

```
handle_request(req):
  try:
    return route(req)
  catch err:
    id = new_correlation_id()
    log.error("unhandled", request_id=id, error=err.detail, trace=err.trace)
    return Response(500, {code: "internal_error",
                          message: "Something went wrong.",
                          request_id: id})
```

**Not covered here**: validation errors that help legitimate users fix input (for example "email format invalid") are fine when they reveal nothing about internals or other users' data. Keep them specific to the user's own input.

**Applies when** an error message crosses a trust boundary to an untrusted or semi-trusted caller. Internal service-to-service errors may carry more detail, but still avoid secrets and treat the caller's trust level honestly.
