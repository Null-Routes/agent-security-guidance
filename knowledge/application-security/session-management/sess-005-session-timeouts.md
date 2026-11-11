---
id: SESS-005
title: Enforce idle and absolute session timeouts proportionate to risk
type: pattern
strength: recommended
evidence: established
category: session-management
summary: Expire sessions after a period of inactivity and after a maximum total lifetime, both enforced on the server, with shorter limits and re-authentication for higher-risk applications and sensitive actions.
lifecycle: [design, implementation, review]
standards: [CWE-613, NIST-SP-800-63B, OWASP-ASVS:V7]
tags: [session-timeout, idle-timeout, absolute-timeout, reauthentication, session-lifetime]
contexts:
  - Choosing session lifetime for a new application
  - Adding "remember me" or long-lived sign-in
  - Protecting sensitive actions such as payment or credential changes
relationships:
  - type: related_to
    target: SESS-003
    note: Expiry is a form of invalidation and must be enforced server-side.
  - type: related_to
    target: AUTH-002
    note: Step-up authentication can be required for sensitive actions.
  - type: related_to
    target: SESS-008
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
  - https://pages.nist.gov/800-63-3/sp800-63b.html
  - https://cwe.mitre.org/data/definitions/613.html
last_reviewed: "2026-11-08"
---

Give every session two limits, and enforce both on the server: an **idle timeout** (no activity for N minutes) and an **absolute timeout** (maximum age since sign-in, regardless of activity). Choose the values from the sensitivity of the data and actions, not from convenience alone.

**Why:** a session left open on a shared or lost device, or a stolen identifier, is useful to an attacker only while it is valid. Shorter lifetimes limit that window; an absolute limit stops a session from living forever through steady use or a stolen, periodically refreshed token.

## Choosing limits

| Risk | Typical approach |
|------|------------------|
| Higher (financial, health, administration) | Short idle timeout (minutes) and an absolute limit of hours; re-authenticate for sensitive actions |
| Moderate (typical business apps) | Idle timeout of tens of minutes; absolute limit of hours to a working day |
| Lower (low-sensitivity consumer content) | Longer lifetimes, with "remember me" reduced in privilege |

Treat the numbers as starting points to adjust with the product owner. NIST SP 800-63B ties reauthentication limits to assurance level, so use it when you have an assurance target. Compliance regimes may prescribe values.

## What to do

- Store creation time and last-activity time in the server-side session and check both on each request. Cookie expiry alone is controlled by the client and is not enforcement.
- Re-authenticate (or step up, [[AUTH-002]]) before sensitive operations such as changing credentials, adding payees or exporting data, even inside a valid session.
- Make "remember me" a separate, lower-privilege, revocable token that cannot perform sensitive actions without re-authentication ([[SESS-003]]).
- For token-based designs, use short access-token lifetimes with refresh limited by the absolute limit ([[SESS-008]]).
- Return a clear signal on expiry and end the session cleanly, discarding server state ([[SESS-003]]).
