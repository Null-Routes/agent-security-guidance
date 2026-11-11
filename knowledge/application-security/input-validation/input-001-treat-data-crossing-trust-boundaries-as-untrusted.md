---
id: INPUT-001
title: Treat all data crossing a trust boundary as untrusted
type: principle
strength: strongly-recommended
evidence: established
category: input-validation
summary: Any data that originates outside the component you control, including headers, files, upstream API responses and database content written by others, may be attacker-influenced. Validate it on entry and encode it on exit.
lifecycle: [design, implementation, review]
standards: [CWE-20, OWASP-Top10-2021:A03, OWASP-ASVS:V2]
tags: [untrusted-input, trust-boundary, validation, injection, taint]
contexts:
  - Designing how a component receives data from users, other services or storage
  - Reviewing code that passes external values to interpreters, file systems or renderers
  - Consuming third-party API responses, webhooks, message queues or uploaded files
relationships:
  - type: supports
    target: INPUT-002
    note: Allow-list validation is the first practical control that follows from this principle.
  - type: supports
    target: INPUT-003
    note: Keeping data separate from commands is how untrusted data is made safe at interpreters.
  - type: supports
    target: INPUT-005
    note: Encoding on output makes untrusted data safe for each destination.
  - type: related_to
    target: ARCH-003
    note: Trust boundaries are identified architecturally; this entry says how data crossing them is treated.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/20.html
  - https://owasp.org/Top10/A03_2021-Injection/
last_reviewed: "2026-11-08"
---

Assume that every value arriving from outside the code you control may have been crafted by an attacker, and decide explicitly what you do with it at the point it enters and at the point it is used. Trust is a property of where data came from, not of how it looks.

## What counts as untrusted

- Everything a client sends: body, query string, path, headers, cookies, file names, uploaded content and metadata, including values your own front end "controls".
- Responses and callbacks from other services, including third-party APIs, webhooks and message queue payloads.
- Data read from stores that other systems or users can write to. A value that was harmless when saved can be dangerous when it reaches a different interpreter later (stored injection).
- Environment-derived input such as file contents, command output, and configuration supplied by tenants or users.

Internal origin is not proof of safety: a compromised or buggy upstream service is also a source of hostile data (see [[ARCH-003]]).

## How to apply it

1. Identify each point where such data enters, and validate it there against what you expect ([[INPUT-002]]).
2. Carry data through the system as data. Never let it become part of a query, command, path or template ([[INPUT-003]]).
3. When emitting it into any context that interprets characters (HTML, URLs, log lines, CSV, shell), encode for that destination ([[INPUT-005]]).
4. Do not mark values "trusted" because an earlier layer looked at them; each interpreter protects itself.

```
value = read_from(boundary)            # untrusted
checked = validate(value, expected)    # reject on failure, at entry
use_in_query(parameter = checked)      # data stays data
emit_to_page(encode_for_html(checked)) # encode at the sink
```

**Applies when** any component accepts input from outside its own trust domain, which is nearly every application. Do not try to detect "malicious-looking" content as the main defence; define what is acceptable instead.
