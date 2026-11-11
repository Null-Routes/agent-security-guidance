---
id: LOG-002
title: Do not write sensitive data to logs
type: anti-pattern
strength: strongly-recommended
evidence: established
category: logging-monitoring
summary: Logging passwords, tokens, keys, session IDs or personal data copies them into stores with weaker protection and longer retention. Log stable identifiers and redacted values instead.
lifecycle: [implementation, review, maintenance]
standards: [CWE-532, OWASP-Top10-2021:A09]
tags: [logging, sensitive-data, secrets, pii, redaction, anti-pattern]
contexts:
  - logging request or response bodies, headers or query strings
  - logging exceptions that embed input values
  - debug logging left enabled in production
relationships:
  - type: violates
    target: DATA-001
    note: Logging extra personal data collects and retains more than the purpose requires.
  - type: related_to
    target: DATA-005
    note: Log files are a common route to exposure of sensitive information.
  - type: related_to
    target: CRYPTO-002
    note: Secrets that leak into logs defeat the purpose of a managed secret store.
  - type: related_to
    target: LOG-001
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/532.html
last_reviewed: "2026-11-07"
---

Do not log credentials, secrets, tokens, full payment or identity data, or other sensitive personal data. Logs are usually widely readable, copied to third-party tooling, backed up and retained longer than the data they quote, so anything written there is effectively disclosed (CWE-532).

**Negative example**

```
# Dumps whole objects and raw requests into the log
log.info("login attempt: " + request.body)           # contains password
log.debug("calling payment API with headers " + headers)  # contains bearer token
log.error("failed for user " + user.toString())      # serializes every field
```

**Why it fails**: the log now holds live credentials and personal data. Anyone with log access, or any later log leak, gains them, and deletion or rotation obligations apply to every copy.

**Corrected approach**
- Log an allow-list of named fields, not whole objects or raw requests.
- Log identifiers (user ID, order ID, correlation ID) instead of the data they point to.
- Mask or drop known-sensitive keys centrally in the logging layer (for example `password`, `authorization`, `token`, `cookie`) so one forgotten call does not leak.
- Never put sensitive values in URLs, since URLs are logged by default ([[DATA-006]]).
- Disable verbose or debug logging in production, and review it before enabling temporarily.
- Treat logs as data subject to classification and retention rules ([[DATA-002]], [[DATA-004]]).

```
log.info("login failed", user_id=lookup_id, request_id=ctx.id, reason="bad_credentials")
```

**Applies when** any code writes diagnostic or audit output. If a value is needed for investigation, log a salted hash or truncated form only when that is genuinely sufficient. Redaction filters are a safety net, not a replacement for choosing fields deliberately.
