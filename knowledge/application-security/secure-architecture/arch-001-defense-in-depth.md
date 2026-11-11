---
id: ARCH-001
title: "Defense in depth: layer independent controls"
type: principle
strength: strongly-recommended
evidence: established
category: secure-architecture
summary: Do not let one control be the only thing between an attacker and an asset. Combine independent preventive, limiting and detective layers so that a bypass or bug in one layer does not become a breach.
lifecycle: [architecture, design, review]
standards: []
tags: [defense-in-depth, layered-security, blast-radius, containment, detection]
contexts:
  - Designing protection for sensitive data or privileged operations
  - Reviewing a design that relies on a single gateway, filter or check
  - Deciding which additional controls are worth their cost
relationships:
  - type: related_to
    target: ARCH-003
    note: Trust boundaries are the natural places to put each layer.
  - type: related_to
    target: AUTHZ-002
    note: Least privilege limits what a compromised component can reach.
  - type: related_to
    target: ARCH-008
    note: Obscurity is acceptable only as an extra layer, never as the control.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Secure_Product_Design_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/218/final
last_reviewed: "2026-11-08"
---

Design every important protection so that it still holds when another control fails. Assume each individual control will eventually have a bug, a misconfiguration or a bypass, and make sure a second, independent control sits behind it.

## Three kinds of layer

- **Prevent:** authentication, authorization, input validation, network segmentation.
- **Limit:** least privilege ([[AUTHZ-002]]), scoped credentials, tenant isolation, rate limits ([[API-002]]), separate stores for separate sensitivity levels. These cap the blast radius when prevention fails.
- **Detect and respond:** security logging and alerting ([[LOG-007]]), anomaly signals, the ability to revoke credentials and roll back quickly.

## How to apply

1. For each high-value asset, list the controls that protect it. If the list has one item, add another layer of a different kind.
2. Make layers independent: they should not share the same code path, credential or failure mode. Two checks that call the same helper are one layer.
3. Keep each layer correct on its own. Do not drop a check because "the gateway already does it"; the service behind it should still authorize ([[ARCH-005]]).
4. Place layers at trust boundaries ([[ARCH-003]]).

```
request -> network rule -> authN -> authZ at service -> schema validation
        -> parameterized query -> least-privilege DB role -> audit log + alert
```

Example: for server-side request forgery ([[API-006]]) combine destination allow-listing in code with network egress rules, so a flaw in either is not fatal.

## Limits

Layers add cost and complexity. Do not stack redundant copies of the same weak control, and do not use "we have layers" to excuse a known flaw in one of them. Fix the flaw.
