---
id: DATA-006
title: Putting sensitive values in URLs
type: anti-pattern
strength: strongly-recommended
evidence: established
category: data-protection
summary: URLs are logged, stored in browser history, sent in referrer headers and shared, so tokens, credentials and personal data placed in them leak widely. Send such values in headers or request bodies instead.
lifecycle: [design, implementation, review]
standards: [CWE-598]
tags: [url, query-string, tokens, referrer, logging]
contexts:
  - Designing links for password reset, email verification, invitations and sharing
  - Authenticating API calls or downloads
  - Redirects and callbacks between systems
relationships:
  - type: violates
    target: DATA-001
    note: Spreads sensitive data into places that have no need to hold it.
  - type: related_to
    target: DATA-005
  - type: related_to
    target: LOG-002
    note: Access logs and proxies record URLs by default.
  - type: related_to
    target: AUTH-005
    note: Reset links must carry a token, so contain its exposure.
  - type: related_to
    target: API-008
sources:
  - https://cwe.mitre.org/data/definitions/598.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html
last_reviewed: "2026-11-08"
---

Do not carry secrets or personal data in the URL, whether in the path or the query string. Even over encrypted connections, the URL is treated as non-secret by most of the surrounding system.

## Negative example

```
# BAD: credential and personal data in the query string
GET /api/reports?api_key=EXAMPLE-NOT-A-REAL-KEY&email=person@example.test&ssn=000-00-0000

# BAD: long-lived session or reset token in a link
https://app.example.test/reset?token=EXAMPLE-LONG-LIVED-TOKEN
```

Where it leaks: web server, proxy, load balancer and CDN logs; monitoring and analytics tools; browser history, bookmarks and autofill; the `Referer` header sent to third-party sites and embedded resources; shared or screenshotted links; chat previews that fetch the URL automatically. Anyone with the link, and every system that recorded it, gets the value.

## Corrected approach

- **Credentials:** put API keys and tokens in an authorization header, and sessions in cookies ([[API-008]]), never in the URL.
- **Personal or sensitive fields:** send them in a request body using POST, or look them up server-side by an opaque identifier ([[DATA-001]]).
- **Links that must carry a secret** (reset, verification, invitation, one-time download): use a high-entropy single-use token that expires quickly, exchange it on arrival for a session or form, and invalidate it at once ([[AUTH-005]]). Better, have the landing page post it onward and redirect to a URL without it.
- **Limit leakage on those pages:** set a restrictive referrer policy, avoid third-party content on them, and mark them non-cacheable.
- **Server logging:** do not log full query strings for sensitive routes, or mask known parameter names ([[LOG-002]]).
- **Identifiers in paths** are not secrets either: authorize object access instead of hiding the ID ([[AUTHZ-003]]).

```
POST /api/reports        Authorization: Bearer <token>
body: { "email": "person@example.test" }
```
