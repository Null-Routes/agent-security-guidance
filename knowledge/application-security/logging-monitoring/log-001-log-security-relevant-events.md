---
id: LOG-001
title: Log security-relevant events with accountable context
type: pattern
strength: strongly-recommended
evidence: established
category: logging-monitoring
summary: Record authentication, authorization, input-rejection and administrative events as structured entries that say who did what, to which resource, from where, when, and with what outcome, so incidents can be investigated.
lifecycle: [design, implementation, review]
standards: [CWE-778, OWASP-Top10-2021:A09, NIST-SP-800-92]
tags: [logging, audit, accountability, security-events, forensics, structured-logging]
contexts:
  - designing or reviewing any service that authenticates users or enforces access control
  - adding administrative or privileged operations
  - preparing a system for incident response
relationships:
  - type: supports
    target: LOG-007
    note: Detection and alerting can only work on events that were recorded in the first place.
  - type: related_to
    target: LOG-002
    note: Context must be rich enough to investigate but must not include secrets or excess personal data.
  - type: related_to
    target: AUTH-003
    note: Failed and throttled authentication attempts are core events to record.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
  - https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/
  - https://cwe.mitre.org/data/definitions/778.html
  - https://csrc.nist.gov/pubs/sp/800/92/final
last_reviewed: "2026-11-07"
---

Write a log entry for every security-relevant decision or state change, and give each entry enough context to reconstruct what happened. Without these records, attacks go unnoticed and investigations cannot establish scope or attribute actions to an actor (CWE-778).

**Record at least these event classes**
- Authentication successes and failures, logouts, credential and MFA changes, account recovery.
- Authorization denials and access to high-value resources.
- Input validation failures that look like probing, and unexpected application errors.
- Privileged and administrative actions, configuration and permission changes.
- Security-control changes such as disabled logging or altered retention.

**Include accountable context in each entry**
- Timestamp in a single unambiguous format (UTC, ISO 8601).
- Actor: authenticated user or service identity, plus a session or request correlation ID.
- Source: client address and user agent where meaningful.
- Action, target resource identifier, and outcome (allowed, denied, failed).
- Reason code for denials, not the full request.

Prefer structured output (key/value or JSON) over free text so entries can be queried and alerted on. Use stable event names.

```
log.event(
  event="authz.denied",
  actor_id=principal.id,
  request_id=ctx.request_id,
  source_ip=ctx.client_ip,
  resource="invoice:8841",
  action="read",
  outcome="denied",
  reason="not_owner"
)
```

Log identifiers, not secrets or whole payloads; see [[LOG-002]]. Send entries to a location the application cannot alter ([[LOG-003]]) and make sure something reads them ([[LOG-007]]).

**Applies when** the system has users, privileges or data worth protecting. Scale volume and retention to risk; logging every field of every request is not the goal.
