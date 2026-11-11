---
id: THREAT-003
title: Choose a threat identification method (STRIDE, attack trees, abuse cases, LINDDUN, CAPEC and ATT&CK)
type: decision-framework
strength: recommended
evidence: established
category: threat-modeling
summary: Pick how you will enumerate threats based on the question you need answered, from per-element categories (STRIDE), attacker goals (attack trees), misuse of features (abuse cases), privacy harms (LINDDUN), or catalogued attack and adversary behaviour (CAPEC, ATT&CK).
lifecycle: [design, architecture]
standards: [OWASP-Top10-2021:A04]
tags: [stride, attack-trees, linddun, capec, attack, abuse-cases, methodology, decision]
contexts:
  - Starting a threat model and unsure how to structure the threat search
  - Adding privacy or detection requirements to an existing model
relationships:
  - type: depends_on
    target: THREAT-002
    note: All methods need a description of the system to work against.
  - type: related_to
    target: THREAT-004
    note: Abuse cases are one of the methods and have their own template.
  - type: related_to
    target: THREAT-001
  - type: related_to
    target: THREAT-005
    note: Output of any method feeds the treatment decision.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
  - https://owasp.org/www-community/Threat_Modeling_Process
  - https://capec.mitre.org/
  - https://attack.mitre.org/
last_reviewed: "2026-11-08"
---

No method is complete; each shapes what you notice. Choose by the question, combine at most two, and prefer the one the reviewer already knows.

| Method | Question it answers | Fits | Watch out for |
|--------|--------------------|------|---------------|
| STRIDE (spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege) | What can go wrong at each element or flow? | Default for a data-flow model ([[THREAT-002]]); easy to apply systematically | Can produce long, generic lists; pair with prioritisation ([[THREAT-005]]) |
| Attack trees | How could an attacker achieve this specific goal? | A few high-value goals (steal data, move money); showing alternative paths | Scales poorly across a whole system |
| Abuse cases | How could legitimate features be misused? | Business logic, automation, authorized users acting badly ([[THREAT-004]]) | Depends on imagination about attacker motive |
| LINDDUN (linkability, identifiability, non-repudiation, detectability, disclosure of information, unawareness, non-compliance) | What privacy harms arise from processing personal data? | Systems handling personal data; complements STRIDE rather than replacing it | Needs clear data-flow and purpose information ([[DATA-001]]) |
| CAPEC | Which known attack patterns apply to this component? | Prompting for threats you would otherwise miss; linking to weaknesses and tests | A catalogue, not a method; many entries will not apply |
| ATT&CK | How do real adversaries behave once in, and could we detect it? | Detection and response planning; high-risk environments ([[LOG-007]]) | Oriented to enterprise and infrastructure behaviour more than application design |

## Practical default

For most features: describe the system ([[THREAT-002]]), apply STRIDE to each boundary-crossing flow, add abuse cases for features that move value or data, add LINDDUN if personal data is involved, and use CAPEC entries to check you have not missed a pattern relevant to a component.

Evidence for these methods is mostly practitioner experience and published guidance rather than controlled studies; STRIDE and attack trees are long established, the others are widely used. Do not treat any as a guarantee of coverage.

## Agent note

State which method you used and what you did not cover (for example "privacy threats not assessed"), so reviewers know the limits of the result.
