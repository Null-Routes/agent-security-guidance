---
id: LOG-007
title: Detect and alert on suspicious behaviour
type: pattern
strength: recommended
evidence: established
category: logging-monitoring
summary: Define a small set of actionable detections over security events, such as repeated failures or authorization denials, route alerts to an owner, and rehearse the response. Collecting logs nobody reads does not detect attacks.
lifecycle: [design, deployment, maintenance]
standards: [OWASP-Top10-2021:A09, CWE-778, NIST-SP-800-92]
tags: [monitoring, alerting, detection, incident-response, anomaly, metrics]
contexts:
  - operating internet-facing or multi-user services
  - defining alert rules and on-call ownership
  - building detections from application events
relationships:
  - type: depends_on
    target: LOG-001
    note: Detections need well-defined, trustworthy events.
  - type: depends_on
    target: LOG-003
    note: Alerts rely on logs that are complete and tamper-resistant.
  - type: supports
    target: AUTH-003
    note: Monitoring authentication failures is half of throttling and monitoring.
  - type: supports
    target: ARCH-001
    note: Detection is a layer that limits the damage of failed preventive controls.
sources:
  - https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/
  - https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/92/final
last_reviewed: "2026-11-07"
---

Turn recorded events into alerts that someone owns and will act on. Preventive controls fail or are bypassed; timely detection limits dwell time and damage. Logging without review is a recognised failure category in its own right.

**Start with high-signal detections**
- Bursts of failed logins, many accounts from one source, or one account from many sources ([[AUTH-004]]).
- Repeated authorization denials or sequential access to many object identifiers ([[AUTHZ-004]]).
- Validation failures or error spikes at a single endpoint.
- Use of privileged functions outside normal hours or by unusual identities.
- New administrator creation, permission changes, disabled logging or security settings.
- Access to decoy data (honey records or tokens) that no legitimate flow touches.
- Sudden volume changes in data export or API usage.

**Make alerts operable**
- Assign each alert an owner, severity and a short response runbook; prune alerts that nobody acts on.
- Tune thresholds against a baseline, and track false positives so alert fatigue does not hide real signals.
- Include correlation IDs and actor context so responders can pivot quickly ([[LOG-001]]).
- Alert on the monitoring pipeline itself going silent ([[LOG-003]]).
- Test detections with simulated attack activity, and rehearse the incident process.
- Automated response such as blocking or throttling should be reversible and rate-limited to avoid being turned into a denial-of-service lever.

```
rule "auth-failure-burst":
  when count(event="authn.failed", by=source_ip, window=5m) > 20
  then alert(severity="medium", runbook="RB-auth-abuse")
```

**Applies when** the system is exposed to untrusted users or holds sensitive data. Scale ambition to risk: a handful of well-owned alerts beats a large unreviewed rule set. Detection complements, and does not replace, prevention.
