---
id: INPUT-004
title: Query injection (SQL, NoSQL, LDAP and similar)
type: vulnerability
strength: strongly-recommended
evidence: established
category: input-validation
summary: Untrusted input that is mixed into query text or query objects can change the query's meaning, exposing or altering data. Recognise string-built queries and pass-through query operators, and fix them with bound parameters.
lifecycle: [implementation, review, testing]
standards: [CWE-89, CWE-90, CWE-943, CAPEC-66, OWASP-Top10-2021:A03, OWASP-ASVS:V1]
tags: [sql-injection, nosql-injection, ldap-injection, query, injection]
contexts:
  - Code that builds queries for relational, document, graph, directory or search backends
  - Filtering, sorting and search features driven by request parameters
  - Reviewing database access layers and reporting endpoints
relationships:
  - type: related_to
    target: INPUT-003
    note: Parameterization is the primary prevention.
  - type: related_to
    target: INPUT-002
  - type: related_to
    target: LOG-005
    note: Verbose database errors help attackers refine injection payloads.
  - type: related_to
    target: AUTHZ-002
    note: A least-privileged database account limits what a successful injection can reach.
sources:
  - https://cwe.mitre.org/data/definitions/89.html
  - https://capec.mitre.org/data/definitions/66.html
  - https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
  - https://owasp.org/Top10/A03_2021-Injection/
last_reviewed: "2026-11-08"
---

Query injection occurs when input changes the structure of a query rather than just supplying a value. Consequences include reading other users' records, bypassing logins, modifying or deleting data and, on some backends, executing code.

## How to recognise it

- A query is built with string concatenation, interpolation or formatting that includes a request-derived value, even after "escaping".
- Identifiers, sort fields or operators come directly from the request.
- A document-store filter is built from a parsed request body, so a caller can supply operator objects (such as "not equal" or "matches pattern") where a plain string was expected.
- Directory filters (LDAP-style) or search-engine query strings include raw user text.
- Second-order cases: a value was stored safely, then later read back and concatenated into another query.
- Error messages from the data store reach the client ([[LOG-005]]).

```
# Vulnerable shape
filter = "(&(uid=" + input + ")(active=true))"       # input can close and extend the filter
query  = { "user": body.user, "pass": body.pass }    # body.pass may be an operator object, not a string
```

## Prevent it

1. Use bound parameters or the store's structured query builders ([[INPUT-003]]); this is the primary fix.
2. Enforce types on input before it reaches the query, so an expected string cannot arrive as an object or array ([[INPUT-002]]).
3. Map dynamic identifiers and sort options to a server-side allow-list.
4. Run with a least-privileged account: read-only where possible, no schema or administrative rights ([[AUTHZ-002]]).
5. Return generic errors to clients and keep detail in logs ([[LOG-004]]).
6. Test with negative cases: values containing quotes, operators and structural characters should be treated as plain data.

Escaping by hand and blocking keywords are fragile and are not substitutes for parameterization.
