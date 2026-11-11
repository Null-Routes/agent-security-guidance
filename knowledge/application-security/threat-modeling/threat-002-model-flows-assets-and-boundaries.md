---
id: THREAT-002
title: Model the system with data flows, entry points, assets and trust boundaries
type: pattern
strength: strongly-recommended
evidence: established
category: threat-modeling
summary: Before asking what can go wrong, write down what the system is, meaning its assets, actors, entry points, data flows and the trust boundaries those flows cross. A compact text description is enough and gives every later threat a concrete place to attach.
lifecycle: [architecture, design]
standards: [OWASP-Top10-2021:A04]
tags: [data-flow-diagram, assets, entry-points, trust-boundaries, attack-surface, text-diagram]
contexts:
  - Starting a threat model for a feature or service
  - Documenting an existing system whose flows are unclear
  - An AI coding agent needs a reviewable system description without drawing tools
relationships:
  - type: related_to
    target: ARCH-003
    note: Trust boundaries identified here are where controls are enforced.
  - type: supports
    target: THREAT-001
  - type: related_to
    target: THREAT-003
    note: Most identification methods walk the elements and flows captured here.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
  - https://owasp.org/www-community/Application_Threat_Modeling
last_reviewed: "2026-11-08"
---

Describe the system before analysing it. Capture five things, each as a short list, and keep them in version control next to the design ([[THREAT-001]]).

1. **Assets:** what an attacker would want or what must stay correct (personal data, credentials, money-moving actions, availability of a key function, audit integrity).
2. **Actors:** anonymous users, authenticated users of each role, other tenants, administrators, partner systems, automated jobs, and the attacker.
3. **Entry points:** every place input arrives (endpoints, queues, file drops, webhooks, admin tools, CLI, scheduled imports).
4. **Data flows:** which data moves between which components and stores, and over what channel.
5. **Trust boundaries:** where data or control passes between zones with different trust (internet to service, service to service, tenant to tenant, user to admin, your code to third-party code). See [[ARCH-003]].

## A text form an agent can produce

```text
Assets:      A1 customer profile (PII), A2 session token, A3 order history
Actors:      visitor, customer, support agent, partner system, attacker
Boundaries:  B1 internet|API, B2 API|database, B3 API|payment provider
Flows:
  F1 customer -> API        (B1)  login credentials, order request
  F2 API -> database        (B2)  A1, A3
  F3 API -> payment provider (B3) card token, amount
Entry points: POST /login, POST /orders, webhook /payment-events
Assumptions:  database reachable only from API network segment
```

Diagram syntax (Mermaid or similar) is fine if the team renders it, but the list form diffs and reviews well and does not depend on tooling.

## Tips

- Mark every flow that crosses a boundary; those are where validation, authentication and authorization belong.
- Write assumptions down; a wrong assumption is a common source of missed threats.
- Include data stores and logs as places where assets rest, not just services.
- Describe the system as built or as designed, and flag any part you are guessing at, so a human can correct it.
