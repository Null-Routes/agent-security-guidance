---
id: THREAT-004
title: Write abuse cases alongside use cases
type: pattern
strength: strongly-recommended
evidence: established
category: threat-modeling
summary: For each feature, describe how it could be misused, by authorized users, by automation or through business-logic gaps, in a short actor, goal, abuse, control and test template. This finds threats that input-focused analysis misses.
lifecycle: [design, implementation, testing]
standards: [OWASP-Top10-2021:A04]
tags: [abuse-cases, misuse-cases, business-logic, automation, requirements]
contexts:
  - Specifying a feature that moves money, grants access or exposes data
  - Reviewing a user story that has only a happy path
  - Flows attractive to bots, such as sign-up, checkout, coupons and messaging
relationships:
  - type: related_to
    target: THREAT-006
    note: Each abuse case should end in a requirement and a test.
  - type: related_to
    target: AUTHZ-003
    note: Abuse by authorized users often means accessing records they should not.
  - type: related_to
    target: API-002
    note: Automated abuse of business flows is limited by rate and quota controls.
  - type: related_to
    target: AUTH-004
    note: Credential stuffing is a typical automation abuse case.
  - type: supports
    target: THREAT-003
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Abuse_Case_Cheat_Sheet.html
  - https://owasp.org/API-Security/editions/2023/en/0xa6-unrestricted-access-to-sensitive-business-flows/
last_reviewed: "2026-11-08"
---

Whenever you write or implement a use case, write at least one abuse case next to it. A use case says what a user is supposed to do; an abuse case says what a malicious, careless or automated actor could do with the same feature. Many serious flaws are not malformed input but legitimate operations used in unintended ways, such as a refund repeated, a discount stacked or an export run by the wrong person.

## Template

```text
Abuse case: <short name>
Actor:      <who; include "authorized user", "bot", "competitor", "insider">
Goal:       <what they gain>
Abuse:      <steps using the feature as designed or just outside it>
Impact:     <what is lost if it works>
Control:    <what prevents or limits it, or "none yet">
Test:       <how we will show the control works>
```

Example: *Actor:* registered customer. *Goal:* obtain a promotion repeatedly. *Abuse:* create many accounts, or replay the redeem request concurrently. *Control:* one redemption per verified identity enforced atomically on the server, plus rate limits ([[API-002]]). *Test:* parallel redeem requests yield one success.

## Prompts for finding abuse cases

- What if the caller is authenticated but not entitled to this object or action ([[AUTHZ-003]])?
- What if the step order is changed, skipped or repeated, or two requests race?
- What if this is automated at scale ([[AUTH-004]])? What is the cost per attempt to the attacker, and per attempt to us?
- What if a legitimate user is coerced, malicious or has a stolen session?
- What does the feature reveal through error messages, counts, timing or existence checks?

## For an AI coding agent

Add the abuse cases to the task's design note or pull request, with the control and test for each. Where the control is "none yet", say so plainly and escalate ([[THREAT-005]]). Keep to the few abuse cases that matter for the feature; a list of twenty unprioritised ideas is not helpful.
