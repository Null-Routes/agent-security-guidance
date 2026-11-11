---
id: SESS-007
title: Cross-site request forgery (CSRF)
type: vulnerability
strength: strongly-recommended
evidence: established
category: session-management
summary: When browsers attach credentials such as cookies automatically, another site can trigger state-changing requests as the signed-in user. Defend with anti-CSRF tokens or origin checks, SameSite cookies and safe GET semantics.
lifecycle: [design, implementation, review, testing]
standards: [CWE-352, CAPEC-62, OWASP-ASVS:V7]
tags: [csrf, xsrf, cookies, samesite, anti-csrf-token, origin-check]
contexts:
  - Browser applications that authenticate with cookies or other ambient credentials
  - Forms and APIs that change state (payments, settings, deletes)
  - Browser clients calling cookie-authenticated APIs
relationships:
  - type: related_to
    target: SESS-004
    note: SameSite cookies help but are not a complete defence.
  - type: related_to
    target: API-008
    note: Cookie-based API authentication needs CSRF defences; explicit bearer headers do not.
  - type: related_to
    target: INPUT-006
    note: XSS defeats CSRF tokens; fix both.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
  - https://owasp.org/www-community/attacks/csrf
  - https://cwe.mitre.org/data/definitions/352.html
last_reviewed: "2026-11-08"
---

CSRF occurs when the browser automatically includes the user's credentials (session cookie, HTTP authentication, client certificate) on a request that a malicious page triggered. The server sees an authenticated request and cannot tell the user did not intend it. The attacker cannot read the response, but can change email addresses, transfer funds or alter settings.

## Exposure

You are exposed when authentication is *ambient* (the browser adds it without the page's code) **and** a request changes state. Token-in-header schemes, where script must explicitly attach the credential, are not exposed in the same way, unless the token is also stored in a cookie.

## Defences (combine them)

- **Anti-CSRF tokens:** include an unpredictable per-session (or per-request) token in forms and custom headers of state-changing requests, and verify it server-side. For stateless designs, use a signed double-submit token bound to the session.
- **Origin verification:** check `Origin` (or `Referer`) on state-changing requests and reject mismatches; also consider the `Sec-Fetch-Site` header where available.
- **SameSite cookies:** `Lax` or `Strict` removes most cross-site sends ([[SESS-004]]). Treat it as defence in depth, not the only control.
- **No state change on safe methods:** never alter state in `GET`, `HEAD` or `OPTIONS`.
- **Require a custom header or non-form content type** for API calls, so cross-site forms cannot produce them; combined with strict CORS rules this blocks simple cross-origin requests.
- **Re-authentication** for the most sensitive actions ([[SESS-005]]).

## Do not rely on

Secret cookies alone, only checking the HTTP method (`POST` is forgeable), or the `Referer` header being present. Cross-site scripting bypasses all CSRF defences, so also address [[INPUT-006]].

## Testing

Replay a state-changing request from a different origin without the token, and with an invalid one; expect rejection. Verify that tokens are tied to the session and not reusable across users.
