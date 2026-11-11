---
id: CRYPTO-008
title: Using broken or deprecated cryptographic algorithms and modes
type: anti-pattern
strength: strongly-recommended
evidence: established
category: cryptography-secrets
summary: Algorithms and modes that were once standard but are now broken or disallowed, such as fast hashes for passwords, ECB mode, static IVs or collision-prone hashes for signatures, give a false sense of protection. Check current NIST guidance before choosing.
lifecycle: [design, implementation, review, maintenance]
standards: [CWE-327, CWE-328, CWE-326, OWASP-Top10-2021:A02, NIST-SP-800-131A]
tags: [deprecated-algorithms, weak-hash, ecb, static-iv, md5, sha1, crypto-agility]
contexts:
  - Reviewing legacy code or copied snippets that use encryption or hashing
  - Choosing an algorithm for a new feature
  - Migrating data or tokens protected with an outdated scheme
relationships:
  - type: violates
    target: CRYPTO-001
    note: Vetted does not mean still acceptable; algorithms age out.
  - type: violates
    target: CRYPTO-005
    note: Unauthenticated modes and reused IVs break authenticated encryption.
  - type: related_to
    target: AUTH-001
    note: General-purpose fast hashes are the wrong tool for passwords.
  - type: related_to
    target: CRYPTO-004
    note: The same applies to deprecated TLS versions and cipher suites.
sources:
  - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
  - https://csrc.nist.gov/pubs/sp/800/131/a/r2/final
  - https://cwe.mitre.org/data/definitions/327.html
  - https://cwe.mitre.org/data/definitions/328.html
last_reviewed: "2026-11-08"
---

Do not choose an algorithm, mode or parameter because it appears in old code, tutorials or training examples. Cryptographic guidance changes as attacks improve and hardware gets faster. Check the current NIST guidance (NIST SP 800-131A on transitions and the OWASP cryptographic storage cheat sheet) at the time you choose, and do not copy lists of "good" algorithms from memory, including this entry.

## Negative example

```text
# BAD 1: fast general-purpose hash for passwords
stored = fast_hash(password)

# BAD 2: block cipher in a mode that leaks patterns, with a fixed IV
STATIC_IV = bytes(16)
ciphertext = block_cipher_encrypt(key, plaintext, mode = ECB_or_fixed_iv)

# BAD 3: collision-weakened hash to authenticate data
signature = legacy_hash(document + shared_string)

# BAD 4: key derived from a password with a trivially low work factor
key = derive(password, iterations = 1)
```

Why this fails: fast hashes let an attacker test enormous numbers of password guesses offline; ECB and fixed IVs reveal repeated plaintext and allow block rearrangement; hashes with practical collision attacks cannot support integrity or signature claims; and encryption with no authentication permits undetected tampering. The code still "works" and passes tests, which is why it survives review.

## Corrected approach

- Passwords: a purpose-built password hash ([[AUTH-001]]).
- Encryption: an authenticated encryption scheme with a unique nonce per message ([[CRYPTO-005]]).
- Integrity and signatures: current MAC or signature schemes using a hash that is currently approved, not a legacy one, through a vetted library ([[CRYPTO-001]]).
- Randomness and keys: secure source and managed lifecycle ([[CRYPTO-007]], [[CRYPTO-006]]).
- Transport: current TLS settings ([[CRYPTO-004]]).
- Confirm every choice against current NIST guidance, and flag usages the guidance says to retire.

## Legacy and compatibility

Where a deprecated algorithm is needed to read old data or talk to a legacy peer, isolate it, label it, plan a migration (store an algorithm version with each value, rehash or re-encrypt on use), and record the residual risk ([[THREAT-005]]). Some non-security uses, such as a checksum for accidental corruption, are acceptable but should be named as such so reviewers do not mistake them for security controls.
