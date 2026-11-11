---
id: SESS-001
title: Generate unpredictable session identifiers with a CSPRNG and keep them opaque
type: pattern
strength: strongly-recommended
evidence: established
category: session-management
summary: Create session identifiers from a cryptographically secure random source with ample entropy, make them meaningless to clients, and look up all session data on the server by that identifier.
lifecycle: [design, implementation, review]
standards: [CWE-330, CWE-331, OWASP-ASVS:V7]
tags: [session-id, csprng, entropy, opaque-token, randomness]
contexts:
  - Implementing or replacing session handling
  - Reviewing a home-grown session or token generator
relationships:
  - type: depends_on
    target: CRYPTO-007
    note: The random source requirements are defined there.
  - type: related_to
    target: SESS-002
  - type: related_to
    target: SESS-008
    note: Opaque identifiers are the basis of server-side sessions.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/330.html
  - https://cwe.mitre.org/data/definitions/331.html
last_reviewed: "2026-11-08"
---

Generate each session identifier from a cryptographically secure random number generator ([[CRYPTO-007]]), with enough length that guessing is infeasible, and treat it as an opaque handle: it carries no meaning and is only a key to server-side session data.

**Why:** a session identifier is a bearer credential. Anyone who guesses or predicts a valid one becomes that user. Identifiers built from counters, timestamps, user IDs, hashes of predictable values or a general-purpose PRNG can be predicted from a few observed samples.

## What to do

- Use the platform's secure random facility. OWASP guidance calls for at least 64 bits of entropy from a good CSPRNG; 128 bits or more (for example 16-32 random bytes, encoded) is a common, comfortable choice.
- Do not encode user ID, role, email, creation time or other attributes in the identifier. Keep that data in the server-side store keyed by the identifier.
- Use the framework or a maintained library's session facility where one exists rather than writing your own ([[CRYPTO-001]]).
- Store only what you need to look the session up; consider storing a hash of the identifier server-side so a read-only leak of the session store does not yield usable identifiers.
- Do not accept client-chosen identifiers; if a presented value does not exist, issue a new one ([[SESS-006]]).
- Keep identifiers out of URLs, logs and error messages ([[DATA-006]], [[LOG-002]]); transport them in a protected cookie or header ([[SESS-004]]).

```text
session_id = base64url(secure_random_bytes(32))
store.put(hash(session_id), {user: u, created: now, ...})   # data lives server-side
```

## Applies when

You use server-side sessions with an opaque identifier. Self-contained tokens have different integrity requirements; see [[SESS-008]] and [[AUTH-007]].
