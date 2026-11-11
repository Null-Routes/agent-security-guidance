---
id: INPUT-002
title: Validate input against allow-lists of syntax, type, length and range at the trust boundary
type: pattern
strength: strongly-recommended
evidence: established
category: input-validation
summary: On the server, at the point of entry, accept only values that match an explicit definition of type, format, length and range, after canonicalizing them. Reject everything else rather than trying to clean it.
lifecycle: [design, implementation, testing, review]
standards: [CWE-20, OWASP-ASVS:V2, OWASP-Top10-2021:A03]
tags: [validation, allow-list, schema, canonicalization, length-limits]
contexts:
  - Implementing request handlers, message consumers and file parsers
  - Defining data models and API contracts
  - Reviewing validation that relies on deny-lists or client-side checks
relationships:
  - type: implements
    target: INPUT-001
    note: Concrete step for handling untrusted data at entry.
  - type: depends_on
    target: INPUT-003
    note: Validation is a first layer only; parameterization and output encoding remain necessary.
  - type: related_to
    target: INPUT-005
  - type: related_to
    target: API-005
    note: Explicit request and response schemas are the API form of this pattern.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/20.html
last_reviewed: "2026-11-08"
---

Define what a valid value looks like and accept only that. Run the check on the server, as early as possible, before the value reaches business logic or any interpreter. Client-side checks improve usability but are trivially bypassed and never count as validation.

## Do this

- **Allow-list, not deny-list.** Specify the expected type, format, length and range per field. Lists of "bad" characters or patterns are routinely bypassed with encodings and variants.
- **Canonicalize first.** Decode and normalize (character encoding, Unicode normalization, path or URL decoding) once, then validate the normalized form, and use that same form afterwards. Validating one representation and using another is a classic bypass.
- **Prefer closed sets.** For values drawn from a known set (country, status, sort field), compare against the set and use your own constant, not the supplied string.
- **Bound everything.** Maximum length for strings, numeric ranges, array sizes, nesting depth and total payload size, so oversized input cannot exhaust memory (see [[API-002]]).
- **Parse into typed values.** Convert to integers, dates or identifiers up front and use the parsed value, not the original text.
- **Reject, do not repair.** Fail with a generic client error ([[LOG-004]]) instead of silently stripping characters; stripped input can reassemble into something dangerous.
- **Validate meaning too.** Syntax is not enough: check business rules (an end date after a start date, an amount within the caller's limit) and that referenced objects are ones the caller may use ([[AUTHZ-003]]).

```
function parse_quantity(text):
    if not matches(text, digits_only, max_length = 6): reject
    n = to_integer(text)
    if n < 1 or n > 1000: reject
    return n
```

## Limits

Validation reduces attack surface but does not make data safe for every destination. Free text such as names or comments must still pass through [[INPUT-003]] and [[INPUT-005]]; do not tighten a name field into uselessness to avoid doing so.
