---
id: LOG-005
title: Information exposure through error messages
type: vulnerability
strength: strongly-recommended
evidence: established
category: logging-monitoring
summary: Error output that includes stack traces, queries, file paths, versions or record details gives attackers reconnaissance and sometimes secrets. Recognise it in default error pages, debug modes and raw exception text.
lifecycle: [implementation, testing, review, deployment]
standards: [CWE-209, OWASP-Top10-2021:A05]
tags: [error-messages, information-disclosure, stack-trace, debug-mode, reconnaissance]
contexts:
  - reviewing exception handling and default error pages
  - testing with malformed input
  - deploying with debug or development settings
relationships:
  - type: related_to
    target: DATA-005
    note: A specific case of exposure of sensitive information.
  - type: related_to
    target: LOG-004
    note: LOG-004 is the corresponding preventive pattern.
  - type: related_to
    target: INPUT-004
    note: Database error text revealed to clients often assists injection attacks.
sources:
  - https://cwe.mitre.org/data/definitions/209.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
last_reviewed: "2026-11-07"
---

Error messages become a vulnerability when they return internal detail to someone who should not have it (CWE-209). The data is rarely directly exploitable, but it lowers the cost of every other attack by revealing structure and sometimes live secrets.

**How it arises**
- Unhandled exceptions rendered with default stack traces or debug pages.
- Raw database, parser or upstream-service errors forwarded to the client.
- Messages containing file paths, hostnames, framework or library versions, SQL text, or configuration values such as connection strings.
- Differing messages or status codes that reveal whether a record, user or file exists, or which validation step failed.
- Development or verbose mode shipped to production.
- Different response timing or size on error branches (a related side channel).

**How to recognise it**
- Send malformed input, wrong types, oversized values, missing fields and unusual encodings to each endpoint in a test environment, and read the entire response, including headers.
- Search code for places that return `exception.message`, `str(err)`, or serialized error objects to callers.
- Compare responses for existing and non-existing resources or accounts.
- Check production configuration for debug flags.

**Prevent it**
- Return generic client errors with a correlation ID and keep detail in logs ([[LOG-004]]).
- Use one global handler, and make the production default the non-verbose one ([[ARCH-002]]).
- Make responses uniform where existence is sensitive ([[AUTH-006]]).
- Add automated tests asserting that error responses contain no stack frames, paths or query text.

```
# Vulnerable
return Response(500, {error: err.message, trace: err.stack, query: sql})
```

**Applies when** any error output reaches users, scanners or other services outside the trust boundary.
