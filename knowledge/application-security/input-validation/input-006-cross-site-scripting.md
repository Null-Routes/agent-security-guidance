---
id: INPUT-006
title: Cross-site scripting (XSS)
type: vulnerability
strength: strongly-recommended
evidence: established
category: input-validation
summary: XSS lets attacker-controlled content run as script in another user's browser session, enabling account actions and data theft. Recognise unescaped output and unsafe DOM sinks, and prevent them with contextual encoding.
lifecycle: [implementation, review, testing]
standards: [CWE-79, CAPEC-63, OWASP-Top10-2021:A03, OWASP-ASVS:V1]
tags: [xss, cross-site-scripting, dom, html-injection, content-security-policy]
contexts:
  - Web pages, emails and admin consoles that display user-influenced data
  - Client-side code that writes to the document or evaluates strings
  - Log viewers, support tools and dashboards that render stored data
relationships:
  - type: related_to
    target: INPUT-005
    note: Contextual output encoding is the primary prevention.
  - type: related_to
    target: INPUT-001
  - type: related_to
    target: LOG-006
    note: Log viewers that render HTML turn log injection into XSS.
  - type: related_to
    target: SESS-004
    note: HttpOnly cookies limit what injected script can steal, though not what it can do.
sources:
  - https://cwe.mitre.org/data/definitions/79.html
  - https://capec.mitre.org/data/definitions/63.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html
  - https://owasp.org/Top10/A03_2021-Injection/
last_reviewed: "2026-11-08"
---

Cross-site scripting happens when data that an attacker influences is interpreted by a victim's browser as markup or script. The injected code runs with the victim's session and the site's privileges, so it can perform actions as the user, read page content, and rewrite what they see.

## Forms

- **Stored:** the payload is saved (comment, profile field, support ticket) and served to others later.
- **Reflected:** a request value is echoed in the response (error message, search term).
- **DOM-based:** client-side code reads a source (URL fragment, message, storage) and writes it to an unsafe sink without the server being involved.

## How to recognise it

- Output built by string concatenation into markup, or a "raw/unescaped" rendering switch applied to non-constant data.
- Client code using sinks that parse strings as HTML or script (setting inner HTML, document write, evaluating strings, setting `href` or `src` from data without checking the scheme).
- Untrusted data placed in script blocks, event-handler attributes, unquoted attributes or inline styles.
- Rich-text features that store or render submitted HTML without a sanitizer.
- Admin or support tools that display customer-supplied text.

```
# Vulnerable
element.set_html(request.param("q"))          # parsed as markup
# Safer
element.set_text(request.param("q"))          # treated purely as text
```

## Prevent it

1. Encode every untrusted value for its context at output ([[INPUT-005]]); use auto-escaping templating.
2. Use text-only DOM APIs; for needed HTML, a maintained sanitizer with a strict allow-list.
3. Allow-list URL schemes before placing user-supplied links into `href` or `src`.
4. Add a restrictive Content Security Policy as defence in depth. It limits damage when a mistake occurs but is not a substitute for encoding.
5. Mark session cookies HttpOnly ([[SESS-004]]) and apply [[SESS-007]] defences so injected script has less to steal.
6. Test displays with benign markup characters and confirm they appear as visible text.
