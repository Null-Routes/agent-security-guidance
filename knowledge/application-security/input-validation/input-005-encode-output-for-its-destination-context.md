---
id: INPUT-005
title: Encode or escape output for its destination context
type: pattern
strength: strongly-recommended
evidence: established
category: input-validation
summary: When data is placed into HTML, attributes, scripts, URLs, CSS, logs or similar, apply the encoding that matches that exact context at the point of output, using a vetted library or auto-escaping facility.
lifecycle: [implementation, review, testing]
standards: [CWE-116, CWE-79, OWASP-Top10-2021:A03, OWASP-ASVS:V1]
tags: [output-encoding, escaping, xss, context, templating]
contexts:
  - Rendering user-supplied or stored values in web pages or emails
  - Writing values into logs, CSV exports, URLs or generated documents
  - Building templates or component libraries
relationships:
  - type: mitigates
    target: INPUT-006
    note: Contextual output encoding is the primary defence against cross-site scripting.
  - type: supports
    target: INPUT-001
  - type: related_to
    target: LOG-006
    note: Log formats and log viewers are output contexts too.
  - type: related_to
    target: INPUT-003
    note: Queries and commands use parameters instead of encoding.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/116.html
last_reviewed: "2026-11-08"
---

Encode every untrusted value at the moment it is written into an output, using the encoding rules of that specific destination. The same string needs different treatment in different places, and encoding done too early (at input) or for the wrong context does not protect you.

## Choose the encoding by context

| Destination | What to do |
|-------------|-----------|
| HTML element text | HTML-entity encode |
| HTML attribute value | Attribute-encode and always quote the attribute |
| URL component | Percent-encode the component; validate the whole URL scheme against an allow-list before use |
| JavaScript data | Avoid inlining; pass data as a data attribute or JSON that is parsed, not as code text |
| CSS value | Avoid user input; otherwise restrict to a safe token set |
| Log line or export | Neutralise control characters and delimiters ([[LOG-006]]); neutralise leading formula characters for spreadsheets |
| SQL / OS command | Do not encode; use parameters or argument arrays ([[INPUT-003]]) |

## Do this

- Prefer templating or rendering facilities that escape by default, and treat any "raw" or "unescaped" output switch as a reviewed exception.
- Place untrusted data only in positions the encoder supports. Never put it in script bodies, event-handler attributes, inline styles or unquoted attributes.
- If you must allow rich content (such as limited HTML), pass it through a maintained sanitizer with a strict allow-list, and keep the sanitizer updated.
- Encode once, at output, from the raw stored value. Do not store pre-encoded data, and avoid double encoding.
- Setting the right content type and character set prevents browsers from guessing a different context.

```
page = "<p>Hello, " + html_encode(display_name) + "</p>"
link = "<a href=\"" + attr_encode(validated_https_url) + "\">profile</a>"
```

Encoding handles output safety; it does not replace validation ([[INPUT-002]]).
