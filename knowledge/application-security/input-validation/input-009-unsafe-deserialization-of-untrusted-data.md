---
id: INPUT-009
title: Unsafe deserialization of untrusted data
type: vulnerability
strength: strongly-recommended
evidence: established
category: input-validation
summary: Deserializing data that lets the sender choose object types or trigger behaviour can lead to code execution, tampering or denial of service. Use data-only formats with schema validation and integrity-protect any state that round-trips through clients.
lifecycle: [design, implementation, review]
standards: [CWE-502, OWASP-Top10-2021:A08]
tags: [deserialization, serialization, object-injection, schema, integrity]
contexts:
  - Reading serialized objects from requests, cookies, queues, caches or files
  - Session or state blobs stored on the client
  - Plugin, import and message-processing features
relationships:
  - type: related_to
    target: INPUT-002
    note: Data-only formats with strict schemas are validated like any other input.
  - type: related_to
    target: INPUT-001
  - type: related_to
    target: CRYPTO-001
    note: Use vetted signing or MAC mechanisms for integrity protection.
sources:
  - https://cwe.mitre.org/data/definitions/502.html
  - https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html
  - https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/
last_reviewed: "2026-11-08"
---

Deserialization turns bytes back into in-memory structures. In many formats and runtimes, the byte stream can name which types to create and cause code to run while they are constructed or used. If the stream comes from an untrusted party, the sender controls that behaviour. Typical outcomes are remote code execution, privilege tampering through altered state, and resource exhaustion from deeply nested or oversized input.

## How to recognise it

- A "load", "unserialize" or "read object" call is applied to request bodies, cookies, hidden fields, message payloads or cache entries that others can influence.
- The format records type names or class references that the reader instantiates automatically.
- Application state (roles, prices, identities) is stored in a client-held blob without integrity protection.
- Parsing options enable resolving external references or arbitrary types.

```
# Vulnerable: sender decides what objects exist
obj = deserialize_objects(request.body)

# Safer: data-only format, fixed shape, then explicit construction
data = parse_json(request.body, max_depth = 10, max_size = 1MB)
validate(data, schema)             # known fields, types, ranges
order = Order(data.id, data.qty)   # you decide the type
```

## Prevent it

1. **Do not deserialize untrusted object graphs.** Prefer simple data formats (such as JSON) parsed into plain structures, then build your own types from validated fields ([[INPUT-002]]).
2. **If native serialization is unavoidable,** restrict to an explicit allow-list of permitted types, and never let input choose arbitrary types.
3. **Protect integrity** of serialized state that leaves your control with a signature or MAC verified before parsing, using vetted primitives ([[CRYPTO-001]]). Integrity checks do not make a hostile-but-signed source safe.
4. **Keep state server-side** where practical, so the client holds only an opaque reference.
5. **Cap size, depth and element counts,** and set time limits.
6. **Run deserialization with low privilege** and isolate it, and keep serialization libraries patched ([[ARCH-007]]).
