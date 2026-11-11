---
id: ARCH-004
title: "Fail securely: errors must not grant access or leak state"
type: principle
strength: strongly-recommended
evidence: established
category: secure-architecture
summary: When a security check, dependency or component errors out, default to denying the action and revealing nothing useful. Treat fail-open as an explicit, documented exception, not the result of an unhandled error.
lifecycle: [design, implementation, review, testing]
standards: [CWE-636]
tags: [fail-closed, fail-secure, error-handling, exceptions, availability]
contexts:
  - Writing authentication, authorization or token-validation code
  - Handling timeouts or outages of a policy, identity or logging dependency
  - Reviewing exception handlers around security decisions
relationships:
  - type: supports
    target: AUTHZ-001
    note: Failing closed is deny by default applied to error paths.
  - type: related_to
    target: LOG-004
    note: Client-facing errors should be generic as well as non-permissive.
  - type: related_to
    target: AUTH-007
    note: Any token parsing or validation error is a rejection.
  - type: related_to
    target: LOG-003
    note: Decide deliberately what happens when logging fails.
sources:
  - https://cwe.mitre.org/data/definitions/636.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Secure_Product_Design_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

On any failure in a security-relevant path, deny. An exception, timeout, missing value or unreachable dependency must never be interpreted as "allowed", and the response must not expose internals that help an attacker.

## How to apply

- **Start from denial.** Initialise the decision to deny and set it to allow only on an explicit, successful positive check.
- **Make every error path decide.** Catch failures narrowly around checks; do not wrap a check in a broad handler that continues as if it passed.
- **Treat missing or malformed input as failure:** absent claim, unparseable token, unknown role, null owner, unrecognised setting.
- **Handle dependency outages deliberately.** If the policy service, identity provider or revocation list is unreachable, the default is to refuse the sensitive action. Serve only what is safe to serve without it.
- **Do not leak on failure.** Return a generic client error and log the detail server-side ([[LOG-004]], [[LOG-005]]). Do not leave partial state behind: roll back, release locks, and clear temporary credentials or files.
- **Fail closed in release builds,** not only in tests; do not let debug bypasses survive.

```
allowed = false
try:
    allowed = policy.check(user, action, resource)   # true only on explicit success
except any error:
    log_internal(error)
    allowed = false
if not allowed: return generic_denied()
```

## Availability trade-off

Failing closed can reduce availability. Some systems (safety-critical, emergency access) may need to fail open or degrade. Make that a recorded decision for a specific component, limited in scope and duration and monitored ([[LOG-007]]), never an accident of unhandled exceptions. Where audit logging is mandatory for an operation, see [[LOG-003]].
