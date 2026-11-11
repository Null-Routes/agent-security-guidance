---
id: LOG-003
title: Protect log integrity and availability
type: pattern
strength: strongly-recommended
evidence: established
category: logging-monitoring
summary: Ship logs off the host to append-only or access-restricted storage, limit who can read or change them, and keep them for a defined period so attackers cannot erase evidence and outages do not lose it.
lifecycle: [architecture, deployment, maintenance]
standards: [NIST-SP-800-92, OWASP-Top10-2021:A09]
tags: [logging, integrity, tamper-evidence, retention, centralized-logging, availability]
contexts:
  - deploying services that handle sensitive data or privileged operations
  - designing centralized log collection
  - setting log retention and access policy
relationships:
  - type: supports
    target: LOG-007
    note: Alerting is only trustworthy if the underlying records cannot be silently altered.
  - type: depends_on
    target: LOG-001
    note: There must be meaningful events to protect.
  - type: supports
    target: AUTHZ-002
    note: Applies least privilege to who may read, write and delete logs.
sources:
  - https://csrc.nist.gov/pubs/sp/800/92/final
  - https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
  - https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/
last_reviewed: "2026-11-07"
---

Treat logs as evidence: make them hard to alter or delete after the fact, and make sure they survive failures. An attacker who gains a foothold will often try to edit or erase the trail, and a disk that fills up can silently stop logging.

**Integrity**
- Forward entries in near real time to a separate collector or storage system, so compromising the application host does not give control of its history.
- Give the application write-only (append) access to the log destination. Restrict read access to people and tools with a need, and restrict deletion further.
- Log changes to logging configuration and access to the logs themselves.
- Where the threat model warrants it, add tamper evidence such as hash chaining, signing, or write-once storage.
- Synchronize clocks across components so events can be correlated.

**Availability**
- Set rotation, size limits and retention explicitly. Retention should reflect investigation needs and legal duties, balanced against data minimization ([[DATA-004]]).
- Monitor the logging pipeline: alert when a source goes quiet, a buffer overflows or storage nears capacity.
- Decide deliberately what happens if logging fails. For high-assurance audit events, failing the operation can be correct; for ordinary diagnostics, degrade gracefully and alert ([[ARCH-004]]).
- Back up or replicate logs that serve as audit records.

```
app -> local buffer -> collector (append-only, separate account)
collector -> immutable store (retention: N days, delete requires dual approval)
monitor: alert if source_silent > 5 min or queue_depth > threshold
```

**Applies when** logs support security response, audit or compliance. For small low-risk systems, a managed logging service with basic access controls and retention may be all that is needed.
