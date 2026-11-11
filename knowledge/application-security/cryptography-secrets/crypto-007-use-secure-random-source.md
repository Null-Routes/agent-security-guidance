---
id: CRYPTO-007
title: Use a cryptographically secure random source for security-sensitive values
type: pattern
strength: mandatory
evidence: established
category: cryptography-secrets
summary: Generate tokens, session and reset identifiers, salts, nonces, keys and other unpredictable values with the platform's cryptographically secure random generator. Never use a general-purpose pseudo-random generator, timestamps or counters.
lifecycle: [implementation, review]
standards: [CWE-330, CWE-338, CWE-331, OWASP-ASVS:V11, NIST-SP-800-90A]
tags: [random, csprng, tokens, nonce, entropy, prng, predictability]
contexts:
  - Creating session IDs, password reset or invitation tokens, API keys, CSRF tokens
  - Generating salts, nonces, IVs or encryption keys
  - Reviewing code that calls a general random or time-based function for identifiers
relationships:
  - type: related_to
    target: SESS-001
    note: Session identifiers rely on unpredictable generation.
  - type: related_to
    target: AUTH-005
    note: Reset tokens must be unguessable.
  - type: related_to
    target: AUTH-001
    note: Password salts come from a secure source.
  - type: supports
    target: CRYPTO-006
    note: Key generation depends on good randomness.
  - type: related_to
    target: CRYPTO-001
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
  - https://cwe.mitre.org/data/definitions/330.html
  - https://cwe.mitre.org/data/definitions/338.html
  - https://csrc.nist.gov/pubs/sp/800/90/a/r1/final
last_reviewed: "2026-11-08"
---

Whenever a value must be unguessable by an attacker, draw it from the operating system or language's cryptographically secure random number generator (CSPRNG), through its standard interface. This is mandatory: the output of ordinary pseudo-random generators, clocks, counters and hashes of predictable inputs can be reconstructed from a few observations, which turns a "random" token into a guessable one. The secure option is built into every mainstream platform and costs nothing extra, so there is no reason to use another.

## Which values need it

Session and reset tokens ([[SESS-001]], [[AUTH-005]]), API keys and invitation links, CSRF tokens, password salts ([[AUTH-001]]), nonces and initialization vectors ([[CRYPTO-005]]), cryptographic keys ([[CRYPTO-006]]), one-time codes, and any identifier whose secrecy is part of access control. Public, non-sensitive identifiers (for example a display ordering) do not need it, but if a value protects access, an unguessable identifier does not replace an authorization check ([[AUTHZ-003]]).

## How

- Call the platform's secure generator directly for the number of bytes needed, then encode the bytes for use (hex, base64url). Token entropy matters more than appearance; follow current OWASP guidance for minimum length.
- Prefer library functions that generate whole tokens, keys or UUID-style values securely; check that the specific function you pick is documented as secure, since some UUID versions and "random" helpers are not.
- Do not seed a CSPRNG yourself, and do not fall back to a weaker generator if the secure one fails; fail instead ([[ARCH-004]]).
- Do not truncate to a few characters, and compare secrets with a constant-time comparison.

```text
# Wrong: predictable
token = hex(general_random_int()) + current_time()

# Right: from the OS secure source
token = encode_url_safe(secure_random_bytes(32))
```

## Detecting misuse in review

Search generation code for general-purpose random functions, time, counters, sequence IDs, or hashes of user data used as secrets.
