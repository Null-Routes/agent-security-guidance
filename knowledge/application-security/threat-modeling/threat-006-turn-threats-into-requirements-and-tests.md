---
id: THREAT-006
title: Turn identified threats into security requirements and tests
type: pattern
strength: recommended
evidence: established
category: threat-modeling
summary: Trace each mitigated threat to a concrete requirement, a control in the design and at least one test that fails if the control is removed, and keep the threat model in version control next to the code.
lifecycle: [design, implementation, testing, review, maintenance]
standards: [OWASP-Top10-2021:A04, NIST-SP-800-218]
tags: [traceability, security-requirements, negative-tests, abuse-case-tests, version-control]
contexts:
  - A threat model exists and now needs to influence code and tests
  - Writing acceptance criteria for a security-relevant feature
  - Reviewing whether a past threat model is still reflected in the system
relationships:
  - type: depends_on
    target: THREAT-005
    note: Only threats chosen for mitigation need requirements and tests.
  - type: related_to
    target: THREAT-004
    note: Abuse cases translate directly into negative tests.
  - type: supports
    target: THREAT-001
    note: Keeps the model live and checkable as the system changes.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Abuse_Case_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/218/final
last_reviewed: "2026-11-08"
---

A threat that is only listed in a document protects nothing. For each threat you decided to mitigate ([[THREAT-005]]), write a chain: **threat -> requirement -> control -> test**, and keep the chain visible in the repository.

## Steps

1. **State a requirement** in testable terms: "A caller can read only orders belonging to their own account" rather than "access control must be secure".
2. **Name the control** that satisfies it and where it is enforced, linking to the relevant pattern, for example [[AUTHZ-003]] or [[INPUT-003]].
3. **Write a test that exercises the abuse**, not only the feature: the second tenant requests the first tenant's record and is refused; the replayed token is rejected; the oversized payload is rejected. A good negative test fails when the control is removed.
4. **Link them** with an identifier or a comment, so a reader can go from threat to test and back.

```text
T-07  Customer reads another customer's order        (STRIDE: information disclosure)
REQ   Order read is allowed only for the owning account
CTRL  Object-level check in order service (see AUTHZ-003)
TEST  test_order_read_denied_for_other_account
```

## Keep it current

- Store the threat model in version control beside the design or code, and change it in the same pull request as the design change ([[THREAT-001]]).
- When a test for a security requirement is deleted or weakened, treat that as a signal to revisit the threat.
- Threats treated as accepted or unmitigated need no test, but should be listed with their status so they are not forgotten.

## For an AI coding agent

When you implement a mitigation, add the corresponding negative test in the same change, and mention the threat it covers in the test name or comment. If you cannot write a meaningful test, say so; do not add a test that passes regardless of the control.
