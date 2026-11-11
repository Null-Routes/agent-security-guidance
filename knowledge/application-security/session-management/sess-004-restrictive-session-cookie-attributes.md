---
id: SESS-004
title: Set restrictive cookie attributes for session cookies
type: pattern
strength: strongly-recommended
evidence: established
category: session-management
summary: Mark session cookies Secure, HttpOnly and SameSite, scope them to the narrowest host and path, and give them a bounded lifetime so browsers expose them less and send them only where intended.
lifecycle: [implementation, review, deployment]
standards: [CWE-614, CWE-1004, CWE-1275, OWASP-ASVS:V7]
tags: [cookies, secure, httponly, samesite, cookie-prefix, session-cookie]
contexts:
  - Browser applications that keep the session identifier in a cookie
  - Reviewing cookie configuration or reverse-proxy and framework defaults
relationships:
  - type: related_to
    target: SESS-007
    note: SameSite reduces cross-site requests but does not replace CSRF defences.
  - type: related_to
    target: SESS-001
  - type: related_to
    target: API-008
    note: Cookie-based API authentication relies on these settings.
  - type: related_to
    target: CRYPTO-004
    note: Secure only protects cookies if TLS is used end to end.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/614.html
  - https://cwe.mitre.org/data/definitions/1004.html
last_reviewed: "2026-11-08"
---

Set these attributes on every cookie that carries a session identifier or authentication state:

| Attribute | Effect | Guidance |
|-----------|--------|----------|
| `Secure` | Browser sends the cookie only over HTTPS | Always. Pair with TLS everywhere ([[CRYPTO-004]]). |
| `HttpOnly` | Page scripts cannot read the cookie | Always, so an XSS flaw cannot simply read and exfiltrate the identifier ([[INPUT-006]]). It does not stop script from making authenticated requests. |
| `SameSite` | Limits when the cookie is sent on cross-site requests | `Lax` is a sound default; `Strict` for high-risk apps; `None` only with a documented need (and `Secure`). |
| `Domain` | Which hosts receive the cookie | Omit it so the cookie is host-only; setting it shares the cookie with every subdomain. |
| `Path` | URL prefix that receives the cookie | Narrow when it is meaningful, but it is not a security boundary. |
| `Max-Age`/`Expires` | Persistence | Prefer a session cookie for sensitive apps; otherwise bound it to the absolute timeout ([[SESS-005]]). |

## Further hardening

- Use the `__Host-` name prefix, which makes browsers require `Secure`, `Path=/` and no `Domain`, preventing sibling subdomains from overwriting the cookie.
- Do not store tokens or sensitive data in `localStorage` as a substitute; script-readable storage is exposed to XSS.
- Send a header such as `Cache-Control: no-store` on responses that set or depend on session state.

## Limits

Cookie attributes are defence in depth. `SameSite` is not a complete CSRF fix: same-site attackers, older browsers and some top-level navigations remain, so keep anti-CSRF controls ([[SESS-007]]). `HttpOnly` mitigates cookie theft, not XSS itself. Browsers apply attribute rules slightly differently, so verify behaviour in the target clients.

```text
Set-Cookie: __Host-sid=<opaque>; Secure; HttpOnly; SameSite=Lax; Path=/
```
