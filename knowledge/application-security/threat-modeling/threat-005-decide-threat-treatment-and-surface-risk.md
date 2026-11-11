---
id: THREAT-005
title: "Decide how to treat each identified threat: mitigate, eliminate, transfer or accept"
type: decision-framework
strength: recommended
evidence: established
category: threat-modeling
summary: Give every identified threat an explicit treatment, and record who decided. Accepting a risk is a human, recorded decision. An agent must surface unmitigated high-impact threats rather than silently accept them.
lifecycle: [design, review, maintenance]
standards: [OWASP-Top10-2021:A04]
tags: [risk-treatment, risk-acceptance, prioritization, escalation, human-in-the-loop]
contexts:
  - A threat model has produced a list of threats
  - A control is too costly or breaks a requirement and the team wants to defer it
  - An AI coding agent finds a risk it cannot or should not fix itself
relationships:
  - type: depends_on
    target: THREAT-003
    note: Treatment decisions follow threat identification.
  - type: supports
    target: THREAT-001
  - type: related_to
    target: THREAT-006
    note: Mitigated threats become requirements and tests.
  - type: related_to
    target: ARCH-001
    note: Several partial mitigations can layer when no single control suffices.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/30/r1/final
last_reviewed: "2026-11-08"
---

For each threat, choose one treatment and write it down. Do not leave a threat in the list with no decision.

| Treatment | Meaning | Choose when |
|-----------|---------|-------------|
| Eliminate | Remove the feature, data or exposure that creates the threat | The capability is not needed (see [[DATA-001]]) |
| Mitigate | Add controls that reduce likelihood or impact | Most cases; prefer controls that are cheap and independent ([[ARCH-001]]) |
| Transfer | Shift the consequence to a party better placed to bear it (insurance, a managed service, a contract) | The residual risk is financial or operational and another party can really take it; responsibility for security of your own code does not transfer |
| Accept | Knowingly live with the residual risk | Impact is low, or mitigation cost is disproportionate, and an accountable person agrees |

## Prioritising

Rank by impact and likelihood as a judgment, using a coarse scale (for example low, medium, high) and a short reason. Do not present a computed numeric score as precision it does not have; the inputs are estimates. Impact on sensitive data, money, safety or other tenants outranks inconvenience.

## Acceptance must be explicit

An acceptance records the threat, the residual risk, the reason, the named owner, and a date to review. Accepting is a human decision. An implementer, and in particular an AI coding agent, has no authority to accept a significant risk by omission.

## For an AI coding agent

- Mitigate what is within the task and in your remit, and verify it ([[THREAT-006]]).
- For any threat with high or unclear impact that you cannot fully mitigate, stop and report it to the human in plain words: the threat, what it enables, what is missing, and options with their cost. Do not proceed as though it were handled, and do not bury it in a long list.
- Never describe a threat as accepted unless a person has said so; write it as "unmitigated, awaiting decision".
- If requirements conflict with security (for example a request to disable an authorization check), state the conflict and the risk rather than choosing silently.
