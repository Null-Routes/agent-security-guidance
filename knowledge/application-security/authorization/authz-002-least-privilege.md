---
id: AUTHZ-002
title: Apply least privilege to users, services, tokens and credentials
type: principle
strength: strongly-recommended
evidence: established
category: authorization
summary: Give every identity, whether a person, service, job or token, only the permissions its task needs, for only as long as it needs them, so that a compromise or mistake has limited reach.
lifecycle: [architecture, design, implementation, deployment, maintenance]
standards: [NIST-SP-800-53:AC-6, CWE-250, CWE-269, OWASP-Top10-2021:A01]
tags: [least-privilege, permissions, scopes, service-accounts, blast-radius, rbac]
contexts:
  - Defining roles, scopes or service accounts
  - Issuing API keys, tokens or cloud credentials
  - Running processes, containers or jobs
  - Reviewing permissions that have accumulated over time
relationships:
  - type: related_to
    target: AUTHZ-001
    note: Default deny means no grant without a rule; least privilege keeps each grant narrow.
  - type: related_to
    target: AUTHZ-006
    note: The chosen access model determines how narrow grants can be expressed.
  - type: supports
    target: ARCH-001
    note: Narrow grants limit what a single compromised component can do.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final
  - https://cwe.mitre.org/data/definitions/250.html
last_reviewed: "2026-11-08"
---

Grant the minimum permissions needed for the specific task, scoped to the specific resources, and for the minimum time. Apply this to people, service accounts, background jobs, API keys, access tokens, database users and the operating-system user the process runs as.

**Why:** credentials leak and code has bugs. The permissions attached to the compromised identity set the damage.

## What to do

- Create separate identities per service and per purpose. Do not share one broad credential across components.
- Prefer narrow, task-based roles or scopes over "admin" or wildcard grants. Avoid `*` in actions and resources.
- Scope data access: a reporting job gets read-only access to the tables it uses; a web tier does not hold schema-change rights.
- Make elevated access temporary and requested for a reason, for example time-boxed grants for operations work, instead of standing admin rights.
- Issue tokens with limited scope and short lifetime; downscope when delegating to another service.
- Run processes as unprivileged users with only the filesystem, network and system capabilities they need.
- Review grants periodically and remove unused ones, especially after role changes and departures. Treat permission creep as a defect.

```text
service "report-job":
  allow read  on  analytics.orders, analytics.customers(columns: id, region)
  deny  everything else            # see AUTHZ-001
  credential lifetime: 1 hour
```

## Trade-offs

Very fine-grained permissions raise administration cost and can push teams towards broad exceptions. Start with a few well-chosen roles ([[AUTHZ-006]]) and refine where the sensitivity justifies it. For API credentials see [[API-008]]; for key and secret scope see [[CRYPTO-002]].
