---
id: SESS-006
title: Session fixation
type: vulnerability
strength: strongly-recommended
evidence: established
category: session-management
summary: If a session identifier chosen or learned by an attacker stays valid after the victim authenticates, the attacker shares the victim's authenticated session. Always issue a new identifier at login.
lifecycle: [design, implementation, review, testing]
standards: [CWE-384, CAPEC-61, OWASP-ASVS:V7]
tags: [session-fixation, session-id, login, cookies, vulnerability]
contexts:
  - Any application where a session exists before login
  - Systems that accept session identifiers from URLs or request parameters
  - Applications with cookies shared across subdomains
relationships:
  - type: related_to
    target: SESS-002
    note: Regenerating the identifier at login is the primary fix.
  - type: related_to
    target: SESS-004
    note: Host-only and prefixed cookies make planting harder.
  - type: related_to
    target: SESS-001
sources:
  - https://owasp.org/www-community/attacks/Session_fixation
  - https://cwe.mitre.org/data/definitions/384.html
  - https://capec.mitre.org/data/definitions/61.html
last_reviewed: "2026-11-08"
---

Session fixation lets an attacker *set* the victim's session identifier instead of stealing it. The attacker obtains or invents an identifier, gets the victim to use it, waits for the victim to log in, and then uses the same identifier, which the server now treats as authenticated.

## How it arises

1. The application creates a session for anonymous visitors and keeps the same identifier after login.
2. The attacker plants a known identifier in the victim's browser: via a link with the identifier in the URL, a cookie set from a sibling subdomain or insecure page, or injected markup.
3. The victim authenticates; the server marks that session as logged in without changing the identifier.

Aggravating factors: accepting identifiers from URL or form parameters, accepting identifiers the server never issued (creating a session for any presented value), and cookies scoped to a parent domain.

## Prevention

- Issue a fresh identifier at login and at privilege change, invalidating the old one ([[SESS-002]]). This is the decisive control.
- Accept identifiers only from the protected cookie or header, never from URLs ([[DATA-006]]).
- Reject identifiers the server did not issue; do not create a session for an unknown value ([[SESS-001]]).
- Use host-only cookies with `Secure` and the `__Host-` prefix so other subdomains cannot overwrite them ([[SESS-004]]).
- Apply timeouts so planted identifiers expire ([[SESS-005]]).

## Testing

Capture the pre-login identifier, authenticate, and verify that the post-login identifier differs and that the old one is rejected. Also test whether the app accepts an arbitrary identifier from a query parameter or a made-up cookie value.
