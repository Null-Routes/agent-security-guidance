---
id: ARCH-003
title: Identify trust boundaries and enforce controls at them
type: principle
strength: strongly-recommended
evidence: established
category: secure-architecture
summary: Find the points where data or control passes between components with different trust levels, and authenticate, authorize and validate there. Never assume something is safe because it came from inside your system.
lifecycle: [architecture, design, review]
standards: [NIST-SP-800-218]
tags: [trust-boundary, architecture, data-flow, validation, privilege-separation]
contexts:
  - Drawing or reviewing an architecture or data-flow diagram
  - Adding a new integration, queue, service or user-supplied input path
  - Deciding where a security check belongs
relationships:
  - type: supports
    target: ARCH-001
    note: Boundaries are where independent layers are placed.
  - type: related_to
    target: THREAT-002
    note: Trust boundaries are a core element of a threat-model data-flow description.
  - type: related_to
    target: INPUT-001
    note: Data crossing a boundary is untrusted until checked.
  - type: related_to
    target: API-006
    note: A server fetching user-chosen URLs crosses a boundary toward internal networks.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Secure_Product_Design_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

A trust boundary is any place where data or control moves between parties that deserve different levels of trust. Locate these boundaries first, then put checks on them. Checks placed only at the "front door" miss every other crossing.

## Where boundaries are

- **Process and network:** browser to server, client to API, service to service, server to internal network or cloud metadata, server to third-party API.
- **Privilege:** unauthenticated to authenticated, user to administrator, application to database or operating system.
- **Tenant and organization:** one customer's data and another's ([[AUTHZ-007]]), your code and a vendor's, your code and a dependency or plugin.
- **Data origin:** user input, uploaded files, message queue contents, values read back from a shared database, responses from other systems, and output of automated tools or AI components.

## What to enforce at a crossing

1. **Authenticate** the party on the other side ([[ARCH-005]]).
2. **Authorize** the specific action and resource ([[AUTHZ-005]]).
3. **Validate** inbound data against a strict schema ([[INPUT-002]]) and **encode** outbound data for its destination ([[INPUT-005]]).
4. **Limit** what flows across: minimal fields ([[DATA-001]]), rate limits, scoped credentials.
5. **Log** the crossing where it is security relevant ([[LOG-001]]).

## Common mistakes

- Trusting a value because an upstream component "already validated it". Revalidate what the next component depends on.
- Treating the internal network as one trust zone; a single compromised host then reaches everything.
- Letting client-held state (hidden fields, tokens, flags) decide server-side authorization.

List the boundaries in text or a simple diagram as part of the design, and revisit the list whenever a new integration appears ([[THREAT-001]]).
