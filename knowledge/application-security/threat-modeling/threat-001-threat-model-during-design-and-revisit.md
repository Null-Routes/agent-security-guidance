---
id: THREAT-001
title: Threat model during design and revisit it when the design changes
type: pattern
strength: strongly-recommended
evidence: established
category: threat-modeling
summary: Before building a feature with security impact, write a short threat model that answers what you are building, what can go wrong, what you will do about it, and whether that is enough. Scale effort to the change and revisit it on defined triggers.
lifecycle: [architecture, design, review, maintenance]
standards: [OWASP-Top10-2021:A04, NIST-SP-800-218]
tags: [threat-modeling, secure-design, proportionality, triggers, ai-agent]
contexts:
  - Starting a new service, feature or integration
  - Changing authentication, data handling or external interfaces
  - An AI coding agent is asked to design or implement something with security impact
relationships:
  - type: depends_on
    target: THREAT-002
    note: A model of the system (flows, assets, boundaries) is the input to every threat question.
  - type: related_to
    target: THREAT-003
    note: Choose how you will find threats.
  - type: related_to
    target: THREAT-005
    note: Every identified threat needs an explicit treatment decision.
  - type: related_to
    target: ARCH-003
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/218/final
  - https://owasp.org/Top10/A04_2021-Insecure_Design/
last_reviewed: "2026-11-08"
---

Threat model during design, before the main code is written, and keep the result with the code. A threat model is a short, structured answer to four questions: what are we building, what can go wrong, what are we going to do about it, and did we do a good enough job. Insecure design cannot be fixed by perfect implementation, so the cheapest time to find a missing control is before it is a missing feature.

## Scale the effort

- **Small change, no new exposure** (internal refactor, copy change): no model needed; note "no change to flows, assets or boundaries".
- **Typical feature that touches data or identity:** a half-page text model in the pull request or design note. See [[THREAT-002]] for what to capture and [[THREAT-003]] for finding threats.
- **New system, new trust boundary, or high-impact data:** a fuller model, reviewed by a human with security knowledge.

Do not skip the model because the task is small; write the one-line justification instead.

## Revisit when any of these happens

- A new trust boundary, external interface, integration or third-party dependency appears ([[ARCH-003]]).
- A new class of data is stored or exposed ([[DATA-002]]).
- Authentication, authorization or tenancy logic changes.
- A new kind of caller (anonymous, partner, automated) is allowed.
- An incident, penetration test or vulnerability report shows the model was wrong.

## For an AI coding agent

Produce the model as plain text in the repository (for example a `threat-model.md` beside the design), not as a hidden thought. List assumptions explicitly. Mark which threats you mitigated, and hand unresolved ones to a human rather than deciding silently ([[THREAT-005]]). A model that nobody reads is wasted effort, so keep it short enough to review in minutes.

A threat model is an aid to judgment, not proof of security; it finds only the threats someone thought to ask about.
