---
id: INPUT-003
title: Keep data separate from commands by using parameterized interfaces
type: pattern
strength: mandatory
evidence: established
category: input-validation
summary: Pass untrusted values to databases, shells and other interpreters through interfaces that carry data separately from the command (bound parameters, argument arrays, safe APIs) instead of building command text from strings.
lifecycle: [design, implementation, review, testing]
standards: [CWE-89, CWE-78, CWE-943, OWASP-Top10-2021:A03, OWASP-ASVS:V1]
tags: [injection, parameterized-queries, prepared-statements, argument-arrays, interpreter]
contexts:
  - Building queries for relational, document, directory or search stores
  - Invoking operating system programs
  - Generating any text later executed or interpreted (templates, expressions, scripts)
relationships:
  - type: mitigates
    target: INPUT-004
    note: Bound parameters remove the ability of data to change query structure.
  - type: mitigates
    target: INPUT-007
    note: Argument arrays without a shell keep data from being interpreted as commands.
  - type: supports
    target: INPUT-001
  - type: related_to
    target: INPUT-002
    note: Validation is a complementary first layer, not a replacement.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html
  - https://owasp.org/Top10/A03_2021-Injection/
last_reviewed: "2026-11-08"
---

Never assemble an interpreter's command language by concatenating or formatting untrusted values into a string. Give the interpreter the command structure and the values through separate channels, so values can only ever be data.

**Why this is mandatory:** the technique is long-standing and supported by essentially every database driver and process-launch API, it costs almost nothing to apply, and it removes the entire class of injection at the point where it occurs. The few cases where it cannot be applied (see below) are narrow and have their own safe handling.

## Do this

- **Queries:** write the statement with placeholders and bind values separately. The same applies to document, directory and search queries: build the query as a structured object, not text, and do not pass user-supplied objects or operators straight in.
- **Processes:** call the program directly with an argument list; do not go through a shell ([[INPUT-007]]).
- **Templates and expressions:** pass data as template variables, never as template source or expression text.
- **Stored procedures** help only if they use bound parameters internally.

```
# unsafe: structure and data mixed in one string
run("SELECT * FROM orders WHERE owner = '" + owner_id + "'")

# safe: structure fixed, value bound
run("SELECT * FROM orders WHERE owner = ?", [owner_id])
```

## What placeholders cannot cover

Identifiers (table, column names), sort directions and operators cannot be bound. Map the request to a fixed set of server-defined constants and use only the constant ([[INPUT-002]]). If dynamic text is truly unavoidable, escape it with the destination's own vetted routine and treat the code as high-risk in review.

Parameterization does not make the data itself safe for other destinations: encode for HTML and similar contexts separately ([[INPUT-005]]).
